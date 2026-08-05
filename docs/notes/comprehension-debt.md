# Comprehension Debt Register

Tracks the gap between *what the project has decided* and *what I can defend*.

The project goal is a working board whose implementation I fully understand. An accurate decision I cannot explain still counts as debt, because at bring-up the information needed to debug a board is on the bench, not in a PDF — an undefended decision becomes an unexplainable symptom.

**Status values**
- `earned` — I can state why the design does it this way, what breaks otherwise, and which number drives it, without looking.
- `partial` — I can state some of the three, or I can follow the reasoning but not reproduce it.
- `unearned` — produced for me; I have not verified or internalised it.

**This register is filled in by me, not by Claude.** Claude may add rows for newly-made decisions and may test a claimed `earned` by asking one of the three questions — but it does not set the status. A register written by the AI would repeat the exact failure it exists to catch.

---

## Design notes (`docs/notes/`)

| File | Status | Notes |
|---|---|---|
| `gnss.md` | unearned | AI-produced. Audit claim-by-claim against SAM-M10Q datasheet + Integration Manual UBX-22020019. Known defect: line 40 lists nine GND pins while calling them "ten". |
| `compass.md` | | stub |
| `power.md` | | stub |
| `connector-Interface.md` | | stub |
| `decision.md` | | stub |
| `layout-rules.md` | | empty |
| `calc.md` | | empty |
| `open-questions.md` | | empty |
| `changelog.md` | | empty |

## Locked decisions (`project management/Technical_Brief.md` §2)

Each row needs the same three-question test. The ones marked ⚠ are deviations from a reference design or datasheet recommendation — those carry the most risk if undefended, because there is no reference to fall back on when they misbehave.

| # | Decision | Status | Defend by answering |
|---|---|---|---|
| 1 | 6-pin Basic GPS port | | Why not the 10-pin full port? |
| 2 | Rectangular, 2-layer | | What does the bottom layer do besides return current? |
| 3 | GPS over UART; module I²C left open | | Why is leaving those pins open safe? |
| 4 | Compass address 0x0E via CAD0→GND, CAD1→3V3 | | Where does the ~46 µA leakage come from, and why is it acceptable? |
| 5 | AP2112K-3.3 regulator | | Which two SAM-M10Q specs picked this part over an AMS1117? |
| 6 | ⚠ TVS only — reverse-polarity diode dropped | | What failure mode is being accepted, and why is it acceptable? |
| 7 | 0.22 F supercap + BAT54 + ~100 Ω backup | | Derive the hold time yourself. Why does the BAT54 have to be there? |
| 8 | ⚠ PPS LED buffered by 2N7002 | | What happens if the LED is driven directly from TIMEPULSE? |
| 9 | SM06B-GHS-TB side-entry connector | | Why connectors at both ends of the cable? |
| 10 | ⚠ No on-module I²C pull-ups (DS-009/010 deviation) | | Compute the resulting sink current with the autopilot's 1.5 kΩ. |
| 11 | BOM lives in KiCad symbol fields | | — process decision, low risk |

## Open items this register depends on

- Integration Manual UBX-22020019 is now in `docs/datasheets/` — it settles the decoupling network currently documented in `gnss.md` as "working values, not cited ones."
- Reference designs for diffing: Integration Manual application circuit, Holybro M10 pinouts in `docs/images/`.
