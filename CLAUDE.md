# [CLAUDE.md](http://CLAUDE.md)

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project goal (read first)

**Learning by doing.** The deliverable is *a working board whose implementation the user fully understands* — not a working board. A correct board the user cannot explain is a failed outcome here.

This has a specific consequence: **an artifact produced by Claude that the user does not understand is worth approximately zero**, even when it is accurate. It moves the task list and leaves the actual objective untouched. Optimize for the user's comprehension, not for output volume or speed.

This is a KiCad hardware project, not a software code repo — treat it accordingly.

## Working agreement

### Default: read-only

Do NOT modify any file without explicit permission. This applies especially to KiCad design files (`.kicad_sch`, `.kicad_pcb`, `.kicad_sym`, `.kicad_mod`, `.kicad_pro`) and to documents (datasheets, `docs/`, `README`, etc.).

**Producing the design artifacts is the user's job** — that is where the learning happens. Inspect, explain, verify, and report; ask before editing or creating anything.

### Where Claude sits

The rule of thumb: **if the user would need to defend it with a scope probe in hand, the user writes it.**


| Work                                                                                                  | Claude's role                                                      |
| ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Locating a spec across the datasheets, KiCad mechanics, formatting, restructuring text the user wrote | **Generate freely** — no comprehension value is lost               |
| Design decisions, calculations, the `docs/notes/` content                                             | **Review only** — the user writes it, Claude checks it             |
| Concepts the user will need at bring-up                                                               | **Tutor only** — explain and question; do not produce the artifact |


Do not draft a note, a design decision, or a calculation on the user's behalf, even when asked casually and even when it would be faster. Offer instead to explain the underlying spec, review a draft, or ask the questions that get the user there. If the user explicitly and knowingly asks for a produced artifact anyway, say once that it will add comprehension debt, log it in `docs/notes/comprehension-debt.md`, and then do it — it is their call.

### Definition of done

A note or design decision is done when the user can state, without looking:

1. **Why** the design does it this way,
2. **What breaks** if it is done differently,
3. **Which number or spec** drives it.

File existence is not done. When the user marks something complete, it is reasonable to ask one of the three questions above.

### Learning mechanisms

- **Explain it back.** After each subsystem, the user describes the design in their own words; Claude flags what is wrong, missing, or unsupported by the datasheets. Cheapest comprehension test available — offer it at the end of each subsystem.
- **Derive one number per subsystem.** The user does the arithmetic (ramp window, hold time, sink current). Claude checks the result and the method, and does not pre-empt it with the answer.
- **Diff against known-good.** This project is a *replication* of a well-known combination, not an invention. Comparing against reference designs — the SAM-M10Q Integration Manual application circuit, the Holybro M10 pinouts in `docs/images/` — is an objective check the user can run without expertise, and it turns "I don't understand this" into specific answerable questions. Prefer it over first-principles derivation from the datasheet alone.
- **Comprehension debt register.** `docs/notes/comprehension-debt.md` tracks which notes and locked decisions are *earned* (user can defend them) versus *unearned* (produced for the user). Unearned items are a bring-up risk: at bring-up the needed information is on the bench, not in a PDF, and an undefended decision becomes an unexplainable symptom. Surface the register when related work comes up.



### Review cadence

Two different things, deliberately kept separate:

- **Continuous cheap review (Claude, free, should be constant).** Gates: after the schematic before layout, after placement before routing. Deferring these means practising for months with no correction signal while errors propagate schematic → layout → notes.
- **One-shot expert review (paid or community, held until pre-fab).** Correct to spend once, on a complete design. Do not suggest burning it earlier.

Cost of a fix, for context: schematic stage = minutes; post-layout = days of re-layout; post-fab = a board spin.

### Active review responsibilities

Areas where text-based analysis adds real value:

1. Datasheet ↔ symbol/footprint verification — cross-check pins, electrical types, and land patterns against the source datasheet PDFs.
2. Connectivity / netlist review — pull-ups, decoupling, backup battery, floating pins, and connector pinout vs Pixhawk DS-009/DS-010 and the Holybro M10 reference.
3. Application-circuit & layout-constraint review from datasheets — e.g. SAM-M10Q antenna keep-out / ground-plane rules, IST8310 magnetometer placement constraints.
4. Standards & BOM consistency — pin assignments, signal naming, voltage levels vs the Pixhawk specs.



### Known limits

No visual/layout judgment, cannot render the board, cannot run ERC/DRC, no impedance/EM/thermal simulation. Do not overstate capability here.

## Project Overview

Astrolabe SAM-M10 is a KiCad 10.0 hardware design for a GPS + compass module targeting Pixhawk-compatible autopilot systems. The primary IC is the u-blox SAM-M10Q GNSS module; the compass is the IST8310 3-axis magnetometer. The board connector is a JST GH conforming to the Pixhawk Connector Standard (DS-009) and Pixhawk Autopilot Bus Standard (DS-010).

Architecture, locked decisions, datasheet-driven constraints, and open questions live in `project management/Technical_Brief.md` — the design's source of truth. Schedule and breakdown live alongside it (`WBS_*.md`, `gantt-*.md`).

## Repository Layout

```
hardware/                    KiCad project files (schematic, PCB, project config)
hardware/libraries/
  symbols/                   Custom KiCad symbol libraries (.kicad_sym)
  footprints/                Custom KiCad footprint libraries (.kicad_mod)
docs/datasheets/             Component datasheets + Pixhawk standards + SAM-M10Q Integration Manual
docs/images/                 Reference pinout images (Holybro Standard M10 GPS v1/v2)
docs/notes/                  Design notes by topic — the learning deliverable
mechanical/                  Mechanical design files (enclosures, mounting)
project management/          Technical brief, WBS, Gantt (+ .drawio diagram versions)
release/                     Fabrication outputs (Gerbers, BOM, pick-and-place)
scripts/                     Helper scripts for generating release artifacts
```



## KiCad Workflow

KiCad 10.0 is used. All design work happens through the KiCad GUI — there are no CLI build commands. Key operations:

- **Open project**: `kicad hardware/astrolabe_sam_m10.kicad_pro`
- **Run ERC** (electrical rules check): Inspect → Electrical Rules Checker in Eeschema
- **Run DRC** (design rules check): Inspect → Design Rules Checker in Pcbnew
- **Generate fabrication outputs**: File → Fabrication Outputs in Pcbnew (Gerbers, drill files, BOM, CPL)



## Custom Libraries

Both the symbol and footprint libraries live inside this repo under `hardware/libraries/`. They are not installed globally — KiCad must be pointed at the local paths. If the libraries appear unresolved after cloning, add them via:

- **Symbols**: Preferences → Manage Symbol Libraries → add `hardware/libraries/symbols/`
- **Footprints**: Preferences → Manage Footprint Libraries → add `hardware/libraries/footprints/`



## Key Components and Pinout References

- **SAM-M10Q**: `docs/datasheets/SAM-M10Q-00B.pdf` (UBX-22013293 R04) for electrical specs; `SAM-M10Q_IntegrationManual_UBX-22020019.pdf` **for the reference application circuit** — the integration manual is the primary document for a replication project, since it carries the circuit being replicated.
- **IST8310**: `docs/datasheets/IST8310 3D Magnetometer Datasheet.pdf`
- **SM06B-GHS-TB connector**: `docs/datasheets/GH Connector.pdf` and the Holybro M10 GPS pinout images in `docs/images/` for the target interface



## Standards Compliance

Pixhawk interface standards are the governing documents for pin assignments and electrical requirements:

- `DS-009 Pixhawk Connector Standard.pdf` — physical connector and pin naming
- `DS-010 Pixhawk Autopilot Bus Standard.pdf` — signal definitions and electrical specs

When editing the schematic or adding pins to the connector, cross-reference these documents to ensure compliance.

*Standing process note: datasheets are re-checked for required support components and critical notes at each phase; new findings are surfaced for user decision before being folded into the design.*