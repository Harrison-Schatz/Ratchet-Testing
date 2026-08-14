---
name: sizing-the-tests
description: Use when a queued testing task is claimed and BEFORE any test is written for it — harden, pin, backfill, and characterize alike. Also use mid-task the moment evidence suggests the class is wrong (a "minor" behavior turns out to gate a critical workflow; a pin reveals a bare risk surface), and whenever a NET.md row's class needs revising. Sizing decides the battery depth and proof standard the behavior pays for.
---

# Sizing the Tests

Classify the behavior(s) a task touches into a risk class by looking them up against `RISKS.md`. Classification is mechanical — the judgment already happened, once, on record, when the human approved the risk model. This skill performs a lookup, never a debate.

**Prevents:** failure mode #3 (criticality inversion — battery depth follows consequence to the user, not whichever code was edited last).

## Step 1 — Name the behaviors

List every behavior ID (`<area>/<slug>`) the queued task touches, including check sub-entries (`<area>/<slug>#<check-slug>`) where they exist. A behavior with no `NET.md` row enters the map first — as `gap` or `R0(<reason>)`, via `mapping-the-net` — because a class has nowhere to live outside the map.

## Step 2 — Look up, don't judge

For each behavior, ask one question: **what does its failure mean for the user?** Match that answer against the class criteria and project examples in `RISKS.md`. Rules:

- **The lookup is mechanical.** If you catch yourself weighing arguments instead of matching criteria, stop — that is a criteria gap, not a judgment call you get to make (see Stop conditions).
- **Highest class wins.** A task touching multiple behaviors inherits the highest class among them.
- **A recorded class is reused, not re-derived.** If the `NET.md` row already carries a class and `RISKS.md` has not been amended since, use it. Changing a recorded class happens only through this skill, with a logged reason.

## Step 3 — Read the battery off the class

| | R0 — Accepted | R1 — Minor | R2 — Major | R3 — Critical |
|---|---|---|---|---|
| **Failure means** | no user-visible consequence | degraded but peripheral experience | a primary workflow breaks, workaround exists | user locked out of core value, data loss/corruption, security or money |
| **Battery** | none — a *recorded decision* with reason, not an omission | happy-path smoke + one failure path | happy + failure paths + key edge cases | characterization pin + happy + failure + edges + invariant/property tests where feasible |
| **Proof standard** | n/a | red demo per test | red demo per test | red demo per test **+** membership in every mutation-audit sample (`auditing-the-suite`) |

Red demos are `proving-by-failure`'s mechanics; the battery says how many tests owe one. **"Less testing" is never "no testing" above R0** — R1 exists so peripheral behaviors get proportionate protection instead of zero.

## Step 4 — Record the sizing

One worklog `sizing` entry in `.ratchet-testing/worklog/<task-id>.md` (discipline per `keeping-state`):

```
## [2026-08-14 09:12] <task-id> — sizing
R2. Behaviors: search/stale-results (R2), export/csv-format (R1) — highest wins.
Battery due: happy + failure paths + key edge cases. Criteria: RISKS.md approved 2026-08-01.
```

Then hand back to the spine: BUILD routes the task type to its skill (`using-ratchet-testing` owns the routing).

## Re-sizing mid-task — the escape hatch

Re-run this skill IMMEDIATELY when evidence contradicts the class:

| Trigger | Move |
|---|---|
| A "minor" behavior turns out to gate a critical workflow | up — re-match against `RISKS.md` |
| A pin or characterization reveals an unprotected risk surface | up |
| The "major" behavior turns out to be dev-only tooling | down, with a logged reason — R0 included |

Procedure: **stop work**, append a worklog `escalation` entry (old class → new class, the trigger, the evidence), re-run the lookup against `RISKS.md`, and **keep all work that survives** — batteries nest, so tests built at the old class count toward the new one. Update the `NET.md` row's class with the logged reason (`mapping-the-net`). De-escalation is equally legal with a stated reason, including down to R0 — recorded in `NET.md` as `R0(<reason>)`, never a silent drop.

## Stop conditions

- `RISKS.md` does not exist, or carries no approver + date → STOP. Nothing above R0 classification is legal against an unapproved model; `defining-the-risks` is the only legal move.
- The criteria don't cover this behavior — you're arguing, not matching → STOP. Route to `defining-the-risks` for an amendment. **Never improvise a class.**

## Rationalization check

| Thought | Reality |
|---|---|
| "This is obviously critical, I don't need the file" | The lookup costs one read. "Obviously" is exactly the judgment this skill exists to remove. |
| "The criteria don't quite fit, but R1 feels right" | A felt class is an improvised class. The amendment costs less than a mis-sized battery. |
| "Escalating now wastes the tests I already wrote" | Batteries nest — everything that survives is kept. The escape hatch is built for this. |
| "It's peripheral — skip the battery entirely" | Above R0, less testing is never no testing. R0 is a recorded decision, not an omission. |
