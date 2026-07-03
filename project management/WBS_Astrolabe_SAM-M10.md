# WBS: Astrolabe SAM-M10Q + IST8310 Pixhawk GPS/Compass Module

Work Breakdown Structure derived from `Technical_Brief.md`. Organized around deliverables (design artifacts, verification results, physical outputs), not the chronological phases in the brief. Numbering is decimal-hierarchical; branch depth varies by how much decomposition each area needs.

---

## 1. Design Documentation & Decision Records
- 1.1 Component notes
  - 1.1.1 SAM-M10Q note (fold in MSL 4, 9600-baud default, UBX-22020019 integration-manual citation)
  - 1.1.2 IST8310 note (Set/Reset cap, POR timing, magnetic keep-out)
- 1.2 Subsystem design notes
  - 1.2.1 Power note (single 3.3 V rail, AP2112K, V_IO ramp window, backup domain)
  - 1.2.2 Interfaces note (UART crossover-on-module, I²C pull-up deviation)
  - 1.2.3 Layout note (antenna ground-plane rule, magnetometer placement constraints)
  - 1.2.4 Calcs note (supercap sizing, pull-up sink current, charge time constant)
- 1.3 Tracking records
  - 1.3.1 Decisions log (locked decisions + rationale)
  - 1.3.2 Open-questions log
  - 1.3.3 Changelog

## 2. Component Library & BOM Data
- 2.1 Symbols
  - 2.1.1 Import/create active-device symbols (SAM-M10Q, IST8310, AP2112K-3.3, 2N7002, BAT54)
  - 2.1.2 Import/create connector symbol (SM06B-GHS-TB)
  - 2.1.3 Import/create discrete symbols (TVS, supercap, passives, LEDs, test points)
- 2.2 Footprints
  - 2.2.1 Import/create device & connector footprints
  - 2.2.2 Land-pattern verification against datasheets (SAM-M10Q, IST8310, GH connector)
- 2.3 Symbol ↔ footprint ↔ datasheet cross-check (pin numbers, names, electrical types)
- 2.4 BOM field population in symbol fields (MPN, manufacturer, ratings, DNP)

## 3. Schematic Design
- 3.1 Power supply & input protection (TVS D1, AP2112K-3.3, LDO output/soft-start cap)
- 3.2 GNSS subsystem
  - 3.2.1 SAM-M10Q core + decoupling (10 µF + 100 nF on VCC, 100 nF on V_IO)
  - 3.2.2 Backup domain (0.22 F supercap, BAT54, ~100 Ω charge resistor → V_BCKP)
  - 3.2.3 Antenna net + SAFEBOOT_N / TIMEPULSE node handling
- 3.3 Compass subsystem
  - 3.3.1 IST8310 core + decoupling (100 nF AVDD/DVDD, VPP→DVDD)
  - 3.3.2 Mandatory C1 4.7 µF Set/Reset cap
  - 3.3.3 Address straps for 0x0E (CAD0→GND, CAD1→3V3) + RSTN pull-up
- 3.4 Interface & connector
  - 3.4.1 SM06B-GHS-TB pinout per Pixhawk 6-pin Basic GPS port
  - 3.4.2 UART wiring with crossover on module (FC TX→RXD, TXD→FC RX)
  - 3.4.3 I²C wiring (no on-module pull-ups — intentional DS009/DS010 deviation)
- 3.5 Indicators (power LED; PPS LED buffered by 2N7002 with ~100 kΩ gate pulldown + ~10 k series)
- 3.6 Test points (3V3, GND, TX, RX, SDA, SCL, RESET_N, SAFEBOOT_N, PPS, DRDY)
- 3.7 Net labeling, reference designators & schematic annotation

## 4. Electrical Verification & Open-Question Resolution
- 4.1 ERC clean run
- 4.2 Datasheet-driven cross-checks
  - 4.2.1 Decoupling network vs Integration Manual UBX-22020019
  - 4.2.2 V_IO ramp (25–35000 µs/V) and abs-max limits (≤3.6 V, ±10 mA pins)
  - 4.2.3 IST8310 POR timing (AVDD <0.1 V for ≥10 ms; rail-collapse behavior)
- 4.3 Pixhawk standards compliance (DS-009 pinout/naming, DS-010 signal & electrical specs)
- 4.4 Resolve component-value open questions
  - 4.4.1 TVS part + clamp voltage (≤ LDO abs-max ≈6.5 V)
  - 4.4.2 Supercap charge-resistor value (τ tradeoff)
  - 4.4.3 LED colours, currents & series resistors
- 4.5 Reset-strategy decision (rail-collapse POR sufficiency vs added provision)
- 4.6 Confirm firmware acceptance of compass address 0x0E (ArduPilot/PX4)

## 5. PCB Layout
- 5.1 Board outline & mechanical (dimensions ≈50×50 mm; mounting-hole count 2 vs 4 + placement)
- 5.2 Stackup & ground plane (2-layer; bottom = continuous ground; antenna ground-plane sizing)
- 5.3 Component placement
  - 5.3.1 Magnetometer keep-out from regulator, supercap, LEDs, high-current copper, ferrous parts
  - 5.3.2 Connector position; decoupling proximity (C1 ≤5 mm from IST8310 pin)
- 5.4 Antenna keep-out & RF feed routing
- 5.5 Signal & power routing (5 V input, 3.3 V rail, UART, I²C, backup net)
- 5.6 Silkscreen, polarity marks & assembly markings

## 6. Design Review & Fabrication Outputs
- 6.1 DRC clean run
- 6.2 Final design review (schematic + layout against §3 constraints)
- 6.3 Gerber & drill file generation
- 6.4 BOM export (from KiCad symbol fields)
- 6.5 Pick-and-place / CPL file
- 6.6 Assembly drawing

## 7. Procurement & Sourcing
- 7.1 Source key parts (SAM-M10Q-00B, IST8310, AP2112K-3.3TRG1, SM06B-GHS-TB, ≥3.3 V-rated 0.22 F supercap)
- 7.2 Availability / MOQ / NRND assessment for key parts
- 7.3 Mating cable & connector (GHR-06V-S, 1:1 cable)
- 7.4 MSL handling plan (MSL 4/3 dry storage, bake schedule before reflow)

## 8. Bring-up & Validation Testing
- 8.1 Test-bench setup (u-center 2, 3.3 V USB-UART, USB-I²C adapter, logic analyzer, level shifter as needed)
- 8.2 GNSS bring-up (connect at 9600 baud; sky view, TTFF; hot/warm/cold-start to validate supercap)
- 8.3 Compass bring-up (detect at 0x0E; I²C bus decode)
- 8.4 Power & protection validation (rail voltage, V_IO ramp, POR/brownout behavior)
- 8.5 Final Pixhawk FC sign-off (ArduPilot/PX4: GPS lock + compass detected at 0x0E)
