# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working Agreement (read first)

This is a KiCad hardware project, not a software code repo — treat it accordingly.

- **Read-only by default.** Do NOT modify any file without explicit permission. This applies especially to KiCad design files (`.kicad_sch`, `.kicad_pcb`, `.kicad_sym`, `.kicad_mod`, `.kicad_pro`) and to documents (datasheets, `docs/`, `README`, etc.). Inspect, explain, and report findings — ask before editing or creating anything.
- **Role:** answer questions about project structure and the included documents, and review the design from a Senior Electrical Engineer perspective.
- **Active review responsibilities** (the areas where text-based analysis adds real value):
  1. Datasheet ↔ symbol/footprint verification — cross-check pins, electrical types, and land patterns against the source datasheet PDFs.
  2. Connectivity / netlist review — pull-ups, decoupling, backup battery, floating pins, and connector pinout vs Pixhawk DS-009/DS-010 and the Holybro M10 reference.
  3. Application-circuit & layout-constraint review from datasheets — e.g. SAM-M10Q antenna keep-out / ground-plane rules, IST8310 magnetometer placement constraints.
  4. Standards & BOM consistency — pin assignments, signal naming, voltage levels vs the Pixhawk specs.
- **Known limits** (don't overstate capability here): no visual/layout judgment, can't render the board, can't run ERC/DRC, no impedance/EM/thermal simulation.

## Project Overview

Astrolabe SAM-M10 is a KiCad 10.0 hardware design for a GPS + compass module targeting Pixhawk-compatible autopilot systems. The primary IC is the u-blox SAM-M10Q GNSS module; the compass is the IST8310 3-axis magnetometer. The board connector is an SM10B-GHS-TB (JST GH 10-pin), conforming to the Pixhawk Connector Standard (DS-009) and Pixhawk Autopilot Bus Standard (DS-010).

## Repository Layout

```
hardware/                    KiCad project files (schematic, PCB, project config)
hardware/libraries/
  symbols/                   Custom KiCad symbol libraries (.kicad_sym)
  footprints/                Custom KiCad footprint libraries (.kicad_mod)
docs/datasheets/             Component datasheets (SAM-M10Q, IST8310, GH connector, Pixhawk standards)
docs/images/                 Reference pinout images (Holybro Standard M10 GPS v1/v2)
mechanical/                  Mechanical design files (enclosures, mounting)
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

- **SAM-M10Q**: Custom symbol (`SAM-M10Q.kicad_sym`) and footprint (`SAM-M10Q.kicad_mod`); refer to `docs/datasheets/SAM-M10Q-00B.pdf`
- **IST8310**: Refer to `docs/datasheets/IST8310 3D Magnetometer Datasheet.pdf`
- **SM10B-GHS-TB connector**: Custom symbol and footprint; refer to `docs/datasheets/GH Connector.pdf` and the Holybro M10 GPS pinout images in `docs/images/` for the target interface

## Standards Compliance

Pixhawk interface standards are the governing documents for pin assignments and electrical requirements:
- `DS-009 Pixhawk Connector Standard.pdf` — physical connector and pin naming
- `DS-010 Pixhawk Autopilot Bus Standard.pdf` — signal definitions and electrical specs

When editing the schematic or adding pins to the connector, cross-reference these documents to ensure compliance.
