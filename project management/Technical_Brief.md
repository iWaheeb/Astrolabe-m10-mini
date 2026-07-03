# Technical Brief — Pixhawk GPS + Compass Module

**Project:** SAM-M10Q (GNSS) + IST8310 (magnetometer) peripheral for Pixhawk-standard autopilots
**EDA:** KiCad 10 (schematic + PCB) · **Status:** architecture and schematic-level design locked; PCB-layout decisions open
**Reference docs (in project):** SAM-M10Q datasheet (UBX-22013293 R04) · IST8310 datasheet v1.8 · DS009 Pixhawk Connector Standard · DS010 Pixhawk Autopilot Bus · JST GH connector spec

---

## 1. Architecture

A rectangular 2-layer board exposing the Pixhawk **6-pin Basic GPS port** (JST GH):

```
J1 (JST GH 6p, side-entry)                     3.3 V rail
 1 VCC 5V ──► TVS (D1) ──► AP2112K-3.3 ──┬──► SAM-M10Q  VCC + V_IO
 2 TX  ──────────────────► SAM-M10Q RXD  ├──► IST8310   AVDD + DVDD
 3 RX  ◄────────────────── SAM-M10Q TXD  ├──► LEDs
 4 SCL ──────────────────► IST8310 SCL   └─► supercap (via BAT54) ► V_BCKP
 5 SDA ◄─────────────────► IST8310 SDA
 6 GND
```

- **GPS over UART**, **compass over I²C** — standard Pixhawk convention.
- Single 3.3 V rail feeds everything (VCC = V_IO and AVDD = DVDD ⇒ no sequencing concerns by construction).

## 2. Locked decisions (with rationale)

| Decision | Choice | Why |
|---|---|---|
| Port | 6-pin Basic GPS | No safety switch/buzzer needed |
| Form factor | Rectangular/square, 2-layer | Cost-effective; bottom layer = ground |
| GPS interface | UART only | Pixhawk convention; module's own I²C left open |
| Compass address | **0x0E** via straps CAD0→GND, CAD1→3V3 | Deterministic + ArduPilot/PX4 default; ~46 µA leakage negligible |
| Regulator | **AP2112K-3.3** (SOT-23-5, 600 mA) | Handles 100 mA inrush; soft-start satisfies V_IO ramp; alternates: RT9013-33, AMS1117 |
| Input protection | **TVS only** on 5 V | Reverse diode dropped: GH connector is keyed + genuine cable used; TVS still covers spikes/hot-plug |
| GNSS backup | **0.22 F supercap** (≥3.3 V-rated, 5.5 V typ.) + BAT54 + ~100 Ω | 28 µA backup draw (Table 14) ⇒ ~3 h ⇒ covers the 2–4 h hot-start window; sized up from 0.1 F after user verified the 28 µA figure |
| Indicators | Power LED + **PPS LED buffered by 2N7002** (~100 kΩ gate pulldown) | TIMEPULSE shares node with SAFEBOOT_N and drives only 2 mA — a direct LED could force safeboot at power-up |
| Connector | **SM06B-GHS-TB** side-entry, connector at both ends (GHR-06V-S cable) | Low profile for a puck; serviceable during prototyping |
| I²C pull-ups | **None on module** | DS009/DS010: autopilot provides 1.5 kΩ; adding 4.7 kΩ ⇒ ≈1.14 kΩ ⇒ ~2.9 mA sink, at the I²C limit. *Intentional deviation from the IST8310 reference circuit* |
| Libraries / BOM | Symbols/footprints from SnapEDA/manufacturer; **BOM lives in KiCad symbol fields** (MPN, Manufacturer, ratings, DNP) — no separate BOM file |
| Documentation | `/docs/notes/` topic-named skeletons (components-sam-m10q, components-ist8310, decisions, power, interfaces, layout, calcs, open-questions, changelog); user writes final prose; indicators kept distributed by concern |

## 3. Critical constraints (datasheet-driven)

**SAM-M10Q**
- **V_IO ramp 25–35000 µs/V** — out-of-range ramp can permanently damage the part; LDO output cap keeps it in-window.
- Abs-max: VCC/V_IO/V_BCKP ≤ 3.6 V; V_IO ≤ VCC + 0.3; digital pins ±10 mA.
- **No built-in overvoltage/reverse protection** — external clamping required (the TVS).
- **SAFEBOOT_N must be high/open at boot** (shares node with TIMEPULSE via 1 kΩ) — hence the buffered LED.
- Inrush up to **100 mA**; V_BCKP backup current **28 µA**; backup domain valid down to 1.65 V.
- Antenna spec assumes a **50×50 mm ground plane** — the published sensitivity/TTFF numbers are conditioned on it.
- **Default UART: 9600 baud 8N1**, NMEA out / NMEA+UBX in — bench tools must connect at 9600 first.
- **MSL 4** — short exposed floor life; bake before reflow likely.
- Exact decoupling network: **verify against Integration Manual UBX-22020019** (datasheet omits it; working values 10 µF + 100 nF on VCC, 100 nF on V_IO).

**IST8310**
- **4.7 µF on C1 is mandatory** (Set/Reset function), **≤5 mm** from the pin, max trace width.
- 100 nF on AVDD and DVDD; VPP→DVDD; RSTN pulled to 3V3 (+ test point); DRDY→test point; NC pins open.
- **POR timing:** AVDD must fall **<0.1 V** for **≥10 ms** to guarantee reset; POR up to 50 ms. No FC reset line on the 6-pin port ⇒ keep 3.3 V-rail bulk modest so the rail truly collapses on power-down; brief brownouts may leave the part un-reset.
- Magnetically sensitive — keep away from regulator, supercap, LEDs, high-current copper, ferrous parts.
- **MSL 3**; reflow JESD22-A113 (260 °C peak); ESD ±4 kV HBM built in (no external ESD array fitted — accepted residual).

**Interface**
- **TX/RX crossover is on the module** (cable is 1:1): FC TX (pin 2) → RXD; TXD → FC RX (pin 3).
- GH contacts: 1.0 A / 50 V per pin — ample.

## 4. Key components

SAM-M10Q-00B · IST8310 · AP2112K-3.3TRG1 · SM06B-GHS-TB(LF)(SN) · 5 V TVS (D1, part TBD) · BAT54 · 0.22 F/5.5 V supercap · 2N7002 · 4.7 µF C1 cap · decoupling set (10 µF, 100 nF ×4, 1 µF ×2) · resistors (~10 k gate series, ~100 k pulldown, ~100 Ω charge, ~10 k RSTN, 2× LED) · 2 LEDs · test points (3V3, GND, TX, RX, SDA, SCL, RESET_N, SAFEBOOT_N, PPS, DRDY).

## 5. Open questions

1. **Layout (Phase 4):** board dimensions & antenna ground-plane size (target ≈50×50 mm); mounting holes (2 vs 4) & placement; component placement / connector position.
2. **Component finalization:** TVS part + clamp voltage (≤ LDO abs-max ≈6.5 V); supercap charge-resistor value (100 Ω ⇒ τ ≈ 22 s); LED colours/currents/resistors.
3. **IST8310 reset strategy:** is rail collapse alone sufficient for reliable POR, or add a provision?
4. **Verify-items:** decoupling vs UBX-22020019; 0x0E accepted by target firmware.
5. **Sourcing:** availability/MOQ/NRND for the four key parts + ≥3.3 V-rated 0.22 F supercap.
6. **Assembly:** MSL 4/3 handling plan (dry storage, bake schedule).

## 6. Testing plan

u-blox **u-center 2** via 3.3 V USB-UART (connect at **9600 baud**) for GNSS — sky view, TTFF, hot/warm/cold-start (validates the supercap). USB I²C host adapter (Aardvark / FT232H+pyftdi) for the compass at 0x0E — adapter must supply pull-ups and 3.3 V signaling (an Arduino Uno is 5 V logic: usable only through a PCA9306/BSS138-class level shifter). Logic analyzer for bus decode. **Final sign-off:** real Pixhawk FC (ArduPilot/PX4) — GPS lock + compass detected at 0x0E.

## 7. Next steps

1. Finish writing the `/docs/notes/` prose from the skeletons (fold in §3's newest findings: MSL, 9600 default, integration-manual citation).
2. Complete Phase 1: import SnapEDA/manufacturer symbols + footprints into KiCad 10; populate BOM symbol fields as parts are placed.
3. Phase 2: draw the schematic per §1–§4; Phase 3: ERC against §3.
4. Resolve §5 items 1–2, then Phase 4 layout under the rules in §3; Phase 5 DRC/review; Phase 6 outputs (Gerbers, drill, BOM export, pick-and-place, assembly drawing).

*Standing process note: datasheets are re-checked for required support components and critical notes at each phase; new findings are surfaced for user decision before being folded into the design.*
