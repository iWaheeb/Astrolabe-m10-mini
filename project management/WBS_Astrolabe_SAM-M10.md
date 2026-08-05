# WBS: Astrolabe SAM-M10Q + IST8310 Pixhawk GPS/Compass Module

**Baseline 2 — re-baselined 2026-08-06.** Supersedes the 2026-07-03 baseline, which was grouped by artifact type and measured in continuous calendar days. See `gantt-Astrolabe_SAM-M10.md` for the schedule and the reasons for the reset.

> **Diagram:** `wbs-astrolabe-sam-m10.drawio` — regenerated for Baseline 2, current with this file.

## How this WBS is organized

Work is grouped **by subsystem**, not by artifact type. Each slice is a vertical cut that carries its own note, library parts, schematic section, and comprehension check:

> **learn → write the note → symbol + footprint → schematic section → derive one number by hand → explain it back → mark `earned`**

The previous WBS grouped all documentation, then all library work, then all schematic — which put nine notes ahead of the first placed symbol and deferred every correction signal by months. Slices trade a little KiCad efficiency (the library manager gets opened five times instead of once) for feedback roughly every 1.5 weeks.

**Definition of done** is comprehension, not file existence: for each item, the user can state *why* the design does it this way, *what breaks* otherwise, and *which number or spec* drives it — without looking. Status is tracked in `docs/notes/comprehension-debt.md`.

Two files accumulate continuously rather than appearing as scheduled tasks: every **derive-by-hand** item below writes its working into `docs/notes/calc.md`, and anything unresolved goes into `docs/notes/open-questions.md` as it surfaces *(these were tasks 1.2.4 and 1.3.2 in Baseline 1)*.

Effort is in **engineer-hours**. Planning capacity is **8 h/week**.

---

## 0. Comprehension-debt clearance & foundation → **M1** (14 h)

New work, absent from Baseline 1. The design already carries decisions made without the user's understanding; that debt is a bring-up risk and is retired first.

- 0.1 Audit `docs/notes/gnss.md` claim-by-claim against `SAM-M10Q-00B.pdf` **and** `SAM-M10Q_IntegrationManual_UBX-22020019.pdf`; mark each claim verified / understood / not understood — **4 h**
  - 0.1.1 Resolve the GND-pin defect at `gnss.md:40` (lists nine pins, calls them "ten")
  - 0.1.2 Settle the decoupling network against the Integration Manual's reference circuit — closes the "working values, not cited ones" caveat *(was 4.2.1)*
- 0.2 Diff the design against known-good: Integration Manual application circuit + Holybro M10 pinouts in `docs/images/` — **3 h**
- 0.3 Defend the eleven locked decisions in `Technical_Brief.md` §2; set status in the debt register *(was 1.3.1)* — **3 h**
- 0.4 Test `docs/notes/connector-Interface.md` against the definition of done — it contains user-written content and may already be `earned` — **1 h**
- 0.5 KiCad project skeleton + local symbol/footprint library paths — **3 h**

## 1. Slice — Power → **M2** (10 h)

- 1.1 Write `docs/notes/power.md` (rail, input protection, current budget, sequencing, ramp) *(was 1.2.1)* — **3 h**
- 1.2 AP2112K-3.3, TVS, passives: symbols + footprints + land-pattern check + BOM fields — **3 h**
- 1.3 Schematic: input protection → LDO → 3.3 V rail + decoupling *(was 3.1)* — **2 h**
- 1.4 **Derive by hand:** V_IO ramp window and inrush headroom *(was 4.2.2)* — **1 h**
- 1.5 Resolve TVS part + clamp voltage ≤ LDO abs-max *(was 4.4.1)* — **0.5 h**
- 1.6 Explain-it-back checkpoint; mark `earned` — **0.5 h**

## 2. Slice — GNSS core & backup domain → **M3** (9 h)

Note already exists; it is *earned* in §0 rather than written here.

- 2.1 SAM-M10Q symbol + footprint + land-pattern verification + BOM fields — **4 h**
- 2.2 Schematic: core + decoupling per the §0.1.2-verified network *(was 3.2.1)* — **2 h**
- 2.3 Schematic: backup domain — supercap, BAT54, charge resistor → V_BCKP *(was 3.2.2)* — **1 h**
- 2.4 Schematic: antenna net + SAFEBOOT_N / TIMEPULSE shared node *(was 3.2.3)* — **0.5 h**
- 2.5 **Derive by hand:** supercap hold time and charge τ; resolve the charge-resistor value *(was 4.4.2)* — **1 h**
- 2.6 Explain-it-back; move `gnss.md` to `earned` — **0.5 h**

## 3. Slice — Compass → **M4** (10 h)

- 3.1 Write `docs/notes/compass.md` *(was 1.1.2)* — **3 h**
- 3.2 IST8310 symbol + footprint + land-pattern verification + BOM fields — **3 h**
- 3.3 Schematic: core + decoupling, mandatory 4.7 µF C1, 0x0E address straps, RSTN pull-up *(was 3.3.1–3.3.3)* — **2 h**
- 3.4 POR timing analysis and the reset-strategy decision — is rail collapse sufficient? *(was 4.2.3, 4.5)* — **1 h**
- 3.5 Confirm firmware acceptance of 0x0E on ArduPilot/PX4 *(was 4.6)* — **0.5 h**
- 3.6 Explain-it-back; mark `earned` — **0.5 h**

## 4. Slice — Interface & connector → **M5** (9 h)

- 4.1 Finish `docs/notes/connector-Interface.md` *(was 1.2.2)* — **2 h**
- 4.2 SM06B-GHS-TB symbol + footprint + land-pattern verification + BOM fields — **3 h**
- 4.3 Schematic: connector pinout, UART with crossover on module, I²C without on-module pull-ups *(was 3.4.1–3.4.3)* — **2 h**
- 4.4 **Derive by hand:** I²C sink current against the autopilot's 1.5 kΩ — the number behind the pull-up deviation — **1 h**
- 4.5 DS-009 / DS-010 compliance check *(was 4.3)* — **0.5 h**
- 4.6 Explain-it-back; mark `earned` — **0.5 h**

## 5. Slice — Indicators & test points → **M6** (6 h)

- 5.1 Indicators rationale into `docs/notes/decision.md` — **1.5 h**
- 5.2 2N7002, LEDs, test-point symbols + footprints + BOM fields — **1 h**
- 5.3 Schematic: power LED, PPS LED buffered by 2N7002, test points *(was 3.5, 3.6)* — **2 h**
- 5.4 **Derive by hand:** LED series resistors and the gate network *(was 4.4.3)* — **1 h**
- 5.5 Explain-it-back; mark `earned` — **0.5 h**

## 6. Schematic completion & review gate A → **M7** (6 h)

- 6.1 Net labels, reference designators, annotation *(was 3.7)* — **2 h**
- 6.2 ERC clean run + fixes *(was 4.1)* — **2 h**
- 6.3 **Review gate A** — Claude review of the complete schematic, before any layout — **2 h**

## 7. PCB layout & review gate B → **M8** (24 h)

- 7.1 Board outline & mechanical: dimensions ≈50×50 mm, mounting holes *(was 5.1)* — **3 h**
- 7.2 Stackup & ground plane; bottom layer as the antenna counterpoise *(was 5.2)* — **2 h**
- 7.3 Placement: magnetometer keep-out, antenna clearance, connector position, C1 ≤5 mm *(was 5.3.1, 5.3.2, 5.4)* — **5 h**
- 7.4 **Review gate B** — placement review before routing — **2 h**
- 7.5 Routing: 5 V input, 3.3 V rail, UART, I²C, backup net *(was 5.5)* — **7 h**
- 7.6 Silkscreen, polarity marks, assembly markings *(was 5.6)* — **2 h**
- 7.7 DRC clean run + fixes *(was 6.1)* — **2 h**
- 7.8 Write `docs/notes/layout-rules.md` from what the layout actually enforced *(was 1.2.3)* — **1 h**

## 8. Expert review & rework → **M9** (6 h + ~1 week wait)

The one-shot paid/community review, deliberately spent once on a complete design.

- 8.1 Prepare the review package (schematic PDF, layout, constraint summary) — **1 h**
- 8.2 Submit; wait on turnaround *(was 6.2)* — **0 h**, ~1 week calendar
- 8.3 Rework from findings — **5 h**

## 9. Fabrication release → **M10** (4 h)

- 9.1 Gerbers & drill files *(was 6.3)* — **1 h**
- 9.2 BOM export from symbol fields *(was 6.4)* — **1 h**
- 9.3 Pick-and-place / CPL *(was 6.5)* — **1 h**
- 9.4 Place the fab order — **1 h**

*Then ~2 weeks of fab + shipping lead time — zero effort, real calendar. Absent from Baseline 1.*

## 10. Procurement (parallel, off critical path) — 4 h

Runs against fab lead time, but **order long-lead parts by M8**: the ≥3.3 V-rated 0.22 F supercap and the SAM-M10Q are the two sourcing risks.

- 10.1 Source key parts *(was 7.1)* — **1.5 h**
- 10.2 Availability / MOQ / NRND assessment *(was 7.2)* — **1 h**
- 10.3 Mating cable & connector, GHR-06V-S *(was 7.3)* — **0.5 h**
- 10.4 MSL 4/3 handling plan: dry storage, bake schedule *(was 7.4)* — **1 h**

## 11. Bring-up & validation → **M11** (15 h)

- 11.1 Bench setup: u-center 2, 3.3 V USB-UART, USB-I²C adapter, logic analyzer *(was 8.1)* — **3 h**
- 11.2 GNSS bring-up — connect at 9600 baud, sky view, TTFF, hot/warm/cold start *(was 8.2)* — **4 h**
- 11.3 Compass bring-up — detect at 0x0E, bus decode *(was 8.3)* — **3 h**
- 11.4 Power & protection validation — rail voltage, V_IO ramp on a scope, POR/brownout *(was 8.4)* — **2 h**
- 11.5 Final Pixhawk FC sign-off: GPS lock + compass detected *(was 8.5)* — **3 h**

---

## Effort summary

| | Hours |
|---|---|
| Critical path (§0–§9, §11) | **113 h** |
| Procurement (parallel) | 4 h |
| **Total** | **117 h** |

## Changes from Baseline 1

**Cut** (scope decision: ceremony only — every circuit stays):
- 6.6 Assembly drawing
- 1.3.3 Changelog as scheduled work (the file remains, updated opportunistically)
- Note sections that no design decision depends on

**Distributed into slices rather than batched:** all of old §2 (symbols, footprints, land-pattern verification, BOM field population) now sits inside the slice that needs each part.

**Moved earlier:** old 4.2.1 (decoupling vs UBX-22020019) moves from verification into §0, because the Integration Manual is now in the repo and is the reference circuit this project is replicating — it belongs at the start, not as a late check.

**Added:** §0 debt clearance, fab/shipping lead time, both review gates, per-slice explain-it-back checkpoints and hand-derived calculations.

**Retained in full:** the supercap backup domain (§2.3) and the 2N7002-buffered PPS LED (§5.3). These are the highest-teaching-value circuits on the board; cutting them would buy schedule at the direct expense of the project's purpose.
