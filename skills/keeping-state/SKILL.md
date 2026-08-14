---
name: keeping-state
description: Use whenever something worth surviving the session is learned or decided — a harvest signal, a sizing, a red demo, a quarantine, a blocker — and at every spine boundary (claimed → sized → built → proven → landed). Also use when creating the .ratchet-testing/ directory for the first time, or when unsure what belongs in STATE.md vs the worklog vs NET.md vs FLAKES.md.
---

# Keeping State

The conversation is volatile memory; `.ratchet-testing/` is disk. This is the parent's `keeping-state` discipline pointed at this system's own directory: anything a fresh session needs gets written at the moment it is known — not at the end, because the end is what a dying session never reaches.

**Prevents:** context loss between sessions — outside the spec's #1–#12 table, and the failure the whole state directory exists to defeat. Failure mode #12 (stale trust in the map) is answered by `resuming-work` and `auditing-the-suite`, both of which can only verify what this skill wrote.

## The layout

```
.ratchet-testing/
├── STATE.md      # roster of active testing tasks (one row each) — id, class, phase, NEXT ACTION, pointers — owner: this skill
├── NET.md        # THE asset: behavior → risk class → test pointers → proof pointer → status (protected / suspended / gap / R0+reason) — owner: mapping-the-net
├── RISKS.md      # human-approved risk model: class criteria, project-specific examples, amendment log — owner: defining-the-risks
├── HARVEST.md    # watermark + last-read pointers into .ratchet/ — owner: harvesting-signals
├── FLAKES.md     # quarantine register: entry, verdict pattern, budget clock, disposition — owner: quarantining-the-flake
├── state/        # <task-id>.md — per-task cold-resume snapshot — owner: this skill
├── worklog/      # <task-id>.md — append-only journal: sizings, decisions, surprises, evidence pointers — owner: this skill
├── evidence/     # red demonstrations and mutation-audit reports, by behavior ID — owner: proving-by-failure
└── issues/       # seam requests and durable records readable by the main ratchet (which never has to read them) — owner: requesting-the-seam
```

Task id: `YYYY-MM-DD-<slug>`. Point at the owner; never restate its template. Add `.ratchet-testing/` to version control — state that isn't pushed dies with the laptop.

Write zones and deconfliction: `using-ratchet-testing` / `harvesting-signals`. This system never writes `.ratchet/`.

## What goes where — the only rule you need

- **Acting on it right now?** → the task's `state/<task-id>.md` (+ its roster row)
- **It happened?** → the task's `worklog/<task-id>.md`
- **Protection changed?** → the NET.md row, via `mapping-the-net` — `protected` requires an evidence/ pointer
- **Evidence corrupted (a flake)?** → FLAKES.md, via `quarantining-the-flake`
- Queued-but-unclaimed work lives in HARVEST.md's queue (`harvesting-signals`); the roster holds ACTIVE tasks only.

## STATE.md — the roster (overwrite when the set of active tasks changes)

```markdown
# Ratchet-Testing State — Roster
updated: 2026-08-14 09:12

## Active tasks
| task-id | class | type | phase | step | NEXT ACTION | pointers |
|---|---|---|---|---|---|---|
| 2026-08-14-pin-refresh-204 | R2 | pin | building | 2/4 | Revert fix a1b2c3d with -n; run the pin test; capture the red | state/2026-08-14-pin-refresh-204.md |
```

`class` ∈ R0–R3 (from `sizing-the-tests`); `type` ∈ harden | pin | backfill | characterize | audit; `phase` ∈ sizing | building | proving | recording (the spine beats a claimed task walks).

## state/<task-id>.md — the snapshot (overwrite at every phase boundary and step)

```markdown
# Task state: 2026-08-14-pin-refresh-204
updated: 2026-08-14 09:12
class: R2
type: pin
phase: building        # sizing | building | proving | recording
step: 2 of 4
behaviors: auth/valid-login#pin-refresh-204

## Next action
Revert fix a1b2c3d with -n, run the pin test, capture the verbatim red output.

## Blocked on
(nothing | the specific question + who can answer it)

## Pointers
worklog:  worklog/2026-08-14-pin-refresh-204.md
evidence: evidence/auth--valid-login.md
net rows: NET.md → auth/valid-login
```

"Next action" is the most important line in the system: ONE imperative sentence a stranger could execute. "Continue testing" is a violation. A task with no work left does not idle — it lands (`landing-the-tests`): roster row cleared, state file removed, worklog kept as the permanent record.

## worklog/<task-id>.md — append-only, 1–4 lines per entry

Entry types: `sizing`, `decision`, `surprise`, `evidence`, `escalation`, `blocked`, `done`, `retro` (inherited from the parent) + `harvest`, `proof`, `quarantine`, `disposition` (this system's four). **`proof` entries point at evidence/ files, never paste them** — the verbatim red and green outputs live in `.ratchet-testing/evidence/<area>--<slug>.md` (`proving-by-failure` owns that format), because that is what audits re-run.

```markdown
## [2026-08-14 09:40] 2026-08-14-pin-refresh-204 — proof
Red demo witnessed for auth/valid-login#pin-refresh-204: reverted a1b2c3d, pin went
red; restored clean, suite green. Outputs: evidence/auth--valid-login.md (2026-08-14).
```

## When to write (checklist)

| Moment | Write |
|---|---|
| Task queued (harvest pass) | queue line in HARVEST.md (`harvesting-signals` owns the format) |
| Task claimed from the queue | roster row + `state/<task-id>.md`; worklog `harvest` — the generating signal with its `.ratchet/` source pointer |
| Task sized | worklog `sizing` (one-line format: `sizing-the-tests`); state file: class set, phase → building |
| Test written / battery step done | state file: step tick + fresh NEXT ACTION; worklog `decision` for any non-obvious choice |
| Red demo witnessed | `evidence/` entry (`proving-by-failure`); worklog `proof` pointing at it |
| Sizing contradicted mid-task | worklog `escalation`; re-run `sizing-the-tests` |
| Waiting on a human or a seam | state file `blocked` + worklog `blocked` (seam requests: `requesting-the-seam`) |
| Task landed | worklog `done`; roster row cleared; state file removed — atomically with the NET.md row and evidence (`landing-the-tests`) |
| Quarantine opened | FLAKES.md entry (`quarantining-the-flake`); worklog `quarantine`; NET.md row → suspended (`mapping-the-net`) |
| Quarantine disposed | FLAKES.md disposition; worklog `disposition` |
| Sweep run | dated mutation-audit entries in `evidence/` + worklog `evidence` (`auditing-the-suite`) |

## Stop conditions

- About to write under `.ratchet/` or production source → stop. That is a zone violation, not state-keeping. Untestable without a prod change → `requesting-the-seam`.
- About to flip a NET.md status by hand "while updating state" → stop; NET.md changes go through `mapping-the-net`, and `protected` without an evidence/ pointer is illegal.
- Roster row with no state file, or state file with no row → reconcile before any new work (`resuming-work` has the procedure).

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll write it all up at the end" | Sessions die mid-task. The end is the one moment state cannot count on. |
| "The red output is short — I'll paste it in the worklog" | Proof lives in evidence/, where audits re-run it. The worklog gets a pointer. |
| "It's obvious what I was doing" | To you, now. The reader is a fresh session with zero memory — write the NEXT ACTION it could execute. |
