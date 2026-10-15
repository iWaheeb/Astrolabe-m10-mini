# GNSS - Support Components & Critical Notes

Part: **u-blox SAM-M10Q, ordering code SAM-M10Q-00B, type number SAM-M10Q-00B-01.** Source: Datasheet at `docs/datasheets/SAM-M10Q Datasheet.pdf` (UBX-22013293 R04, 19-May-2023). Integration manual at  `docs/datasheets/SAM-M10Q Integration Manual` (UBX-22020019 R02, 01-Jun-2023).

## Why SAM-M10Q?

- **Antenna is integrated.** 15 x 15 mm patch + Surface Acoustic Wave (SAW) filter that reduces unwanted bands + Low Noise Amplifier (LNA) that boost weak GNSS signals inside the module (Integration Manual, 4.3.2 RF front-end). Having a full package like it make board design easier.
- **Concurrent 4-constellation** (GPS/QZSS L1C/A, Galileo E1-B/C, GLONASS L1OF, BeiDou B1C)  
with SBAS + QZSS enabled by default.



## Role & interface

- Provides position/velocity/time to the flight controller over UART protocol.
- **UART to Flight Controller:** TXD (pin 13) -> connector RX, RXD (pin 14) to connector TX. See [[connector-Interface]](connector-Interface.md).
- **Default UART is 9600 baud, 8N1**,  NMEA out, NMEA + UBX in (Datasheet, Table 16).
- UART baudrate possible configurations: [4800, 9600, 19200, 38400, 57600, 115200, 230400, 460800, 921600] (Integration Manual, Table 11).
- **The module's own I²C (SDA pin 9, SCL pin 12) is left unconnected.** Pixhawk convention drives GPS over UART. Pins are left open, all inputs have internal pull-ups in normal operation (Datasheet, section 5), so open is the datasheet-sanctioned state.



## Power supply & decoupling

- Operating window: `VCC` 2.7–3.0–3.6 V, `V_IO` 2.7–3.0–VCC. Abs max 3.6 V on `VCC`, `V_IO` and `V_BCKP` (Datasheet, Table 11).
- **VCC (pin 17) and V_IO (pin 2) are tied to the same 3.3 V rail** from the AP2112K-3.3. This is deliberate: the absolute-max for `VCC` and `V_IO` is 3.6 V, and more importantly; `V_IO` must not be higher than `VCC` by more than 0.3 V (Datasheet, 4.1 Absolute maximum ratings). Shorting them makes that constraint unviolatable by design. See [[power]](power.md).
- Current draw when the chip is configured to utilize all the four constellations it supports (GPS, Galileo, Glonass, BeiDou) at 3.0 V supply  (typical voltage) with position update rate of 1 Hz, and LNA set to low gain (Datasheet, 4.3 Indicative power requirements):

  | Rail            | Acquisition (search) | Tracking (lock is established) |
  | --------------- | -------------------- | ------------------------------ |
  | Current at VCC  | 13 mA                | 10 mA                          |
  | Current at V_IO | 2.3 mA               | 2.3 mA                         |

- **Inrush at startup can reach 100 mA (Datasheet, 4.3 Indicative power requirements)**. It's the module startup draw; The rail must source it without sagging below the VCC minimum.
- All nine GND pins (1, 4, 5, 6, 10, 11, 15, 16, 20) go to the ground coper (ground plane) through its own via (Datasheet, table 9).



## V_IO voltage-ramp constraint

> `Voltage ramp on V_IO: min 25 µs/V, max 35000 µs/V` — "Exceeding the voltage ramp speed may
> permanently damage the device." (Table 10)
> Read as time-per-volt, not volts-per-time:

- **Faster than 25 µs/V is forbidden.** A 3.3 V rail must take at least **82.5 µs** to rise.
- **Slower than 35000 µs/V is forbidden.** The same rail must be up within **115.5 ms**.
So the legal window for our 0→3.3 V edge is roughly **82.5 µs to 115 ms**.

Consequences for the design:

1. The AP2112K's soft-start (typ. a few hundred µs to ~1 ms to regulation) lands comfortably inside
  the window — this is a stated reason for choosing it over a part with no soft-start.
2. **Do not over-bulk the 3.3 V rail.** A large output capacitor stretches the rise; this pushes
  toward the slow limit, and it also fights the IST8310 POR requirement in the other direction
   ([[compass]](compass.md) wants the rail to *collapse* cleanly on power-down).
3. **No hard hot-plug of the 3.3 V node.** The rail must come up through the LDO, never by shorting
  a charged cap onto it — that is the fast-limit violation.
4. Worth catching on a scope at bring-up: 10–90% rise time of 3V3 at the SAM-M10Q pin.



## Backup supply voltage (V_BCKP)

- **V_BCKP (pin 3) operating range 1.65–3.6 V**; abs max 3.6 V. Below ~1.45 V (V_IOSWITCH) the
internal supply for the backup domain switches over — the backup domain holds RTC and ephemeris
for hot/warm start.
- **Backup current: 28 µA typ.** at V_BCKP = 3.3 V with V_IO = VCC = 0 (Table 14). In normal
operation (everything powered) it is only ~3 µA (fn.26).
- Implementation: **0.22 F supercap** (5.5 V-rated part, must be ≥3.3 V-rated) charged from 3V3
through **~100 Ω** and a **BAT54** isolation diode into V_BCKP.
  - Hold time: usable swing ≈ 3.3 V − V_f(BAT54, µA-level ≈ 0.2–0.3 V) → 1.65 V ⇒ ΔV ≈ 1.35 V.
  `t = C·ΔV / I = 0.22 × 1.35 / 28e-6 ≈ 10600 s ≈ **2.9 h**` — covers the 2–4 h window in which a
  hot start is actually worth having.
  - Charge time constant: `τ = RC = 100 Ω × 0.22 F ≈ 22 s`; the resistor also limits the inrush the
  LDO sees when a fully-discharged supercap is connected at first power-up.
  - The BAT54 stops the supercap from back-feeding the 3.3 V rail (and the rest of the board) after
  power-down; the LDO does not tolerate reverse current from output to input.
- **V_BCKP must never be left floating and must never exceed 3.6 V** — the supercap's 5.5 V rating
is a *cap* rating; the node itself is clamped by the 3.3 V rail it charges from.
- Sizing was raised from 0.1 F to 0.22 F once the 28 µA figure was confirmed against Table 14.



## Strap / control pins


| Pin   | Name       | Treatment                                                                 | Why                                                     |
| ----- | ---------- | ------------------------------------------------------------------------- | ------------------------------------------------------- |
| 7     | TIMEPULSE  | → PPS LED **through a 2N7002**, ~100 kΩ gate pulldown, ~10 kΩ gate series | see below                                               |
| 8     | SAFEBOOT_N | **Leave open** (test point only)                                          | internal pull-up 6/17/72 kΩ; low at start-up = safeboot |
| 18    | RESET_N    | **Leave open** (test point only)                                          | internal pull-up 7/10/13 kΩ; must be low ≥1 ms to reset |
| 19    | EXTINT     | **Leave open**                                                            | unused input, internal pull-up                          |
| 9, 12 | SDA, SCL   | **Leave open**                                                            | module I²C unused (see above)                           |


**The TIMEPULSE / SAFEBOOT_N trap.** Pins 7 and 8 are *the same node*, joined inside the module by a
**1 kΩ series resistor** (Table 9, Table 12). Anything that pulls TIMEPULSE low at power-up pulls SAFEBOOT_N low through that 1 kΩ and **boots the receiver into safeboot mode** — no navigation output, and a confusing failure that looks like a dead module.

## Protection

**The module has none.** Explicitly, sectioh 4.1: *"The product is not protected against overvoltage or
  reversed voltages. Voltage spikes exceeding the power supply voltage specification … must be
  limited to values within the specified boundaries by using appropriate protection diodes."*

- Our protection lives entirely upstream, on the 5 V input: a **TVS on VCC_5V** at the connector,
clamping below the AP2112K's abs-max (~6.5 V). Part and clamp voltage still open — see
[[open-questions]](open-questions.md).
- **Reverse-polarity diode intentionally omitted.** Rationale: the JST GH housing is keyed and a
genuine Pixhawk cable is used, so reversal is a mis-assembly failure, not an operating one; the
TVS still covers spikes and hot-plug transients. Documented as a deliberate deviation in
[[decision]](decision.md).
- Digital pins are limited to **±10 mA** and to `V_IO + 0.3 V` (max 3.6 V). The FC side of the link
is 3.3 V logic per DS-009/DS-010, so no level shifting is needed — but nothing 5 V-tolerant should
ever be assumed here. **UART TX/RX are not 5 V-tolerant.**
- **RF input abs-max 0 dBm at the antenna feed** (Table 10). Relevant only if this board is bench-run
next to a transmitter or a GNSS simulator — do not inject into the patch.



## Layout

Full board rules live in [[layout-rules]](layout-rules.md); the module-specific ones are:

- **Ground plane is the antenna.** section 1.6: the application PCB is part of the antenna solution, and
the specified radiation pattern assumes a **50 x 50 mm ground plane**. Gain and radiation
efficiency fall off on anything smaller. Target board size is set by this, not by component area.
- Bottom layer = **solid, unbroken ground pour** under the module. No signal routing, no splits, no
keep-out slots crossing the region under the patch.
- All ten GND pins stitched to the pour with their own vias, plus a via fence around the module
perimeter.
- **Nothing above the patch.** Keep the sky-facing side clear — no tall parts, no metal, no
fasteners over the antenna, and no ground-plane copper on the top side above the element.
- Keep the switching/noisy neighbours (LDO, LEDs, the supercap charge path) off the antenna side and
away from the module edge; keep them away from the IST8310 too, for a different reason
([[compass]](compass.md)).
- Decoupling caps sit at the pins, ground-side first into the pour.
- The antenna element may show **shallow scratches** from production tuning (section 1.6) — this is normal
and must not be flagged as cosmetic damage at incoming inspection.
- **MSL 4** (section 8.1, IPC/JEDEC J-STD-020): short exposed floor life. Dry-pack storage, and a bake before reflow is likely required. Coordinate with the IST8310's MSL 3 in the assembly plan.
- Operating/storage temperature -40 to +85 °C.

---



### Open verify-items

1. Decoupling network vs **UBX-22020019** (Integration Manual not in repo).
2. TVS part number and clamp voltage.
3. Confirm 3V3 ramp on the bench sits inside 82.5 µs – 115 ms.
4. Supercap availability at ≥3.3 V rating, 0.22 F.

See [[open-questions]](open-questions.md) and [[calc]](calc.md).