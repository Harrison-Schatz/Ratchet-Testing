---
name: mapping-the-net
description: Use whenever protection changes — a test lands, a quarantine strips a verdict of authority, a gap is discovered, a test is deleted — and at the RECORD beat of every spine task. Also use when a behavior needs an ID (a new durable test, a harvested brief acceptance check naming an unmapped behavior), when creating NET.md for the first time, or when unsure whether a missing test is deliberate (R0) or a gap.
---

# Mapping the Net

`NET.md` is THE asset: the map of what the net protects, what it deliberately does not, and where the holes are. The net is measured in behaviors, not lines — a coverage percentage is never a gate anywhere in this methodology.

**Prevents:** failure mode #2 (coverage theater), #8 (test intent illegibility), #9 (unknown blind spots).

## The behavior-ID convention (owned here)

- **Grammar:** `<area>/<slug>`, kebab-case, capability-level — e.g. `auth/valid-login`.
- **Check-level sub-entries:** `<area>/<slug>#<check-slug>` — e.g. `auth/valid-login#expired-token`.
- **In test code:** every durable test carries the tag `[net: <behavior-id>]` in the test name or docstring/comment. A test with no behavior ID is rent-defaulting by definition (`charging-the-rent`).
- **Granularity:** capability rows + check sub-entries (provisional — amend in place with a logged reason). Mirrors brief → acceptance-check structure, so harvested briefs map cleanly onto rows.

## NET.md — the template (non-owners point here; they never restate it)

```markdown
# NET — the behavior map
updated: 2026-08-14

| behavior ID | risk class | test pointers | proof pointer | status |
|---|---|---|---|---|
| auth/valid-login | R3 | tests/auth/test_login.py::test_valid_login | evidence/auth--valid-login.md | protected |
| auth/valid-login#expired-token | R3 | tests/auth/test_login.py::test_expired_token | evidence/auth--valid-login.md | protected |
| search/fresh-results | R2 | — | — | gap |
| tools/log-formatting | R0 | — | — | R0(dev-only tooling, no user-visible consequence) |
```

- **Status vocabulary, exactly:** `protected | suspended | gap | R0(<reason>)`. No other values; no blank cells in the status column.
- **Risk class** comes from RISKS.md via `sizing-the-tests` — classes attach to behaviors, never to tasks; a task inherits the highest class among the behaviors it touches.
- **Proof pointers** resolve into `.ratchet-testing/evidence/<area>--<slug>.md` (behavior ID with `/` flattened to `--`; entry format owned by `proving-by-failure`).

## Row lifecycle

1. **A row enters as `gap` or `R0(<reason>)` — never silently.** The moment a behavior is known (harvested brief, discovered risk surface, seam request), it gets a row. A behavior with no row is the blind spot this file exists to shrink.
2. **`protected` requires a proof pointer into `evidence/`** — a recorded red demonstration per `proving-by-failure`. No proof pointer, no `protected`; without the witnessed failure the status is a claim, not a fact.
3. **Quarantine flips the row to `suspended` the same moment its test loses authority** (`quarantining-the-flake`). The map never overstates the net — not for one commit.
4. **An R2+ `gap` or suspension auto-queues backfill at the next harvest** (`harvesting-signals` → `backfilling-the-gap`). A suspended R2+ behavior is a gap with a history.
5. **Deletion updates the row in the same motion** (`charging-the-rent`, or a flake disposition of delete): the row records the honest resulting state — `gap` or `R0(<reason>)` — never a stale `protected`.
6. **A behavior untestable without a production-source change** stays `gap` with a pointer to its seam request (`requesting-the-seam`). The gap is visible until the seam lands.

## Deliberate vs. accidental — R0 is legal and loud

Deliberate non-protection is minimality; accidental non-protection is a gap. NET.md never confuses the two:

- `R0(<reason>)` requires the reason inline. An R0 with no reason is a gap wearing a costume — reclassify it.
- R0 rows are reviewed at every audit (`auditing-the-suite`); a reason that no longer holds is re-run against RISKS.md via `sizing-the-tests`.
- A behavior nobody ever considered is not R0 — it isn't in the map at all. When discovered, it enters as `gap`, and becomes R0 only through a recorded decision.

## Who writes it, and when

Any spine skill at RECORD — and only at RECORD. Row changes land per `landing-the-tests` atomicity.

The map is a claim until checked: audits (`auditing-the-suite`) mutation-sample rows and re-prove them; resumes (`resuming-work`) verify touched rows against the actual test files. Expect to be audited; write rows you'd survive.

## Growth path

One file until it hurts. If NET.md outgrows comfortable single-file size, it becomes an index over `net/<area>.md` — the index holds one line per area, the rows move into per-area files under the same column and status rules. **Deferred until it hurts** — do not pre-build the index for a map with forty rows.

## Stop conditions

- **No RISKS.md exists** → no risk class can be assigned, so no row can be classified. Stop; `defining-the-risks` is the only legal first task.
- **Tempted to set `protected` without an evidence/ pointer** → structurally blocked. Run `proving-by-failure` first; the status waits for the red.
- **The behavior has no seam to test at** → record the row as `gap`, file the request via `requesting-the-seam`, and stop. Never smuggle a production edit to make a row protectable.

## Rationalization check

| Thought | Reality |
|---|---|
| "Coverage is 90%, the map can wait" | Lines covered is not behaviors protected. The unmapped 10% is where login lives. |
| "I'll update NET.md at the end of the session" | Sessions die mid-task. A stale map claims protection that doesn't exist — failure mode #12, by your own hand. |
| "This behavior obviously needs no test — skip the row" | Then it's an R0 decision: recorded, with a reason. Unrecorded absence is a gap wearing your confidence. |
| "The test exists, so the row is protected" | Existence is not proof. `protected` requires a witnessed red in evidence/. |
