# Schedule: Astrolabe SAM-M10Q + IST8310 Pixhawk GPS/Compass Module

**Baseline 2 — re-baselined 2026-08-06.** Derived from `WBS_Astrolabe_SAM-M10.md` (Baseline 2).

> **Chart:** `gantt-astrolabe-sam-m10.drawio` — regenerated for Baseline 2, current with this file.

## Why Baseline 1 was abandoned

Baseline 1 ran 2026-07-03 → 2026-09-11: 71 continuous calendar days, solo engineer. At the reset it was **46% through the calendar with ~0% earned**. Two assumptions failed:

1. **Capacity.** It assumed continuous days of full attention. Real capacity is **6–10 h/week**.
2. **Measurement.** It counted files, not understanding. Task 1.1.1 produced an accurate note the user could not defend — the burndown moved while the actual objective (a board whose implementation the user understands) did not.

Baseline 2 is measured in **engineer-hours against real capacity**, and completion means *earned*, per `docs/notes/comprehension-debt.md`.

## Planning parameters

| | |
|---|---|
| Start | 2026-08-06 |
| Capacity (planning figure) | **8 h/week** (midpoint of 6–10) |
| Critical-path effort | **113 h** |
| Parallel effort (procurement) | 4 h |
| Non-working calendar | ~3 weeks (fab/shipping ~2 wk, expert-review turnaround ~1 wk) |
| **Projected completion** | **2026-12-10** |

**Sensitivity to capacity:** ~2026-11-19 at 10 h/week · ~2027-01-05 at 6 h/week.

## Milestones

Chart: **`gantt-astrolabe-sam-m10.drawio`** — open in draw.io. The table below carries the same information in text.

| # | Milestone | WBS | Effort | Target |
|---|---|---|---|---|
| M1 | Debt cleared; KiCad project + libraries set up | §0 | 14 h | 2026-08-24 |
| M2 | Power slice `earned` | §1 | 10 h | 2026-09-02 |
| M3 | GNSS + backup slice `earned` | §2 | 9 h | 2026-09-10 |
| M4 | Compass slice `earned` | §3 | 10 h | 2026-09-19 |
| M5 | Interface & connector slice `earned` | §4 | 9 h | 2026-09-27 |
| M6 | Indicators & test points `earned` | §5 | 6 h | 2026-10-02 |
| M7 | **Schematic complete** — ERC clean, review gate A passed | §6 | 6 h | 2026-10-07 |
| M8 | **Layout complete** — DRC clean, review gate B passed | §7 | 24 h | 2026-10-28 |
| M9 | Expert review passed + rework | §8 | 6 h + ~1 wk wait | 2026-11-08 |
| M10 | **Fab released** — Gerbers, drill, BOM, CPL, order placed | §9 | 4 h | 2026-11-12 |
| — | *Boards in hand* | — | 0 h (~2 wk lead) | 2026-11-26 |
| M11 | **Board alive** — GPS lock + compass at 0x0E on a real FC | §11 | 15 h | 2026-12-10 |
| | **Total (critical path)** | | **113 h** | |

Procurement (§10, 4 h) runs parallel from ~2026-10-14 and is off the critical path — but **long-lead parts must be ordered by M8**. The ≥3.3 V-rated 0.22 F supercap and the SAM-M10Q are the two sourcing risks; if either has a real lead time, §10.1 starts earlier and this ordering changes.

**M1 carries ~4 days of deliberate ramp-in buffer** — 14 h at 8 h/week is 1.75 weeks, which would land 08-18. The buffer is there because M1 is the restart milestone after a stall and starts mid-week. No other milestone is buffered.

## Critical path

Foundation → five slices → schematic gate → layout → expert review → fab release → *(fab lead time)* → bring-up.

Everything except procurement is critical, because a solo engineer cannot parallelize effort. **The only lever on the end date is removing work or raising capacity** — adding parallel branches does nothing.

## Velocity tracking

Log actual hours per milestone. This is the stall detection Baseline 1 lacked; a month passed unnoticed because nothing compared planned against actual.

| Milestone | Planned | Actual | Notes |
|---|---|---|---|
| M1 | 14 h | | |
| M2 | 10 h | | |
| M3 | 9 h | | |
| M4 | 10 h | | |
| M5 | 9 h | | |
| M6 | 6 h | | |
| M7 | 6 h | | |
| M8 | 24 h | | |
| M9 | 6 h | | |
| M10 | 4 h | | |
| M11 | 15 h | | |

**Tripwire:** if **M1 slips past 2026-08-31**, the 8 h/week figure is wrong. Re-cut the plan against observed velocity rather than defending the dates. Same rule at every milestone — two consecutive weeks of no progress on an item means it gets cut, split, or handed off.

## Notes on the estimates

- Hours are budgeted against the **comprehension** definition of done, not file creation. Task 1.1.1 came in at 2–3 h against a 2-day budget, but that was AI-produced under the old definition — it is deliberately **not** used to scale this plan down.
- Estimates are recommendations for review, not measured values. The first real data arrives at M1.
- Layout (§7, 24 h) is the largest single block and the least certain, since it is the part of the work with no prior data point.
