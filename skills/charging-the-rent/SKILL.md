---
name: charging-the-rent
description: Use when the suite's cost or clutter is in question — queued by `auditing-the-suite`'s sweep (a runtime-budget breach makes the sweep due, and the sweep queues this census). Also use when a test carries no `[net: ...]` tag, when nobody can say what a test protects, or when the user says "the suite is slow", "clean up the tests", "do we need all these?". Every test pays rent; this skill collects.
---

# Charging the Rent

A net you can't afford to run protects nothing. Every test in the suite pays rent: it maps to a `NET.md` behavior AND contributes unique protection. This skill audits the suite tenant by tenant, evicts the defaulters, and records every eviction — deletion is a first-class outcome, not a confession.

**Prevents:** failure mode #7 (suite cost creep) and failure mode #8 (test intent illegibility).

## When it runs

Queued by `auditing-the-suite` — a runtime-budget breach makes the sweep due, and the sweep queues the census. Also fires on explicit human request, which preempts the queue as always.

Deconfliction check (`harvesting-signals`) before any test-file write.

## The rent test

Two questions per test; both must pass:

1. **Maps** — does it carry a `[net: <behavior-id>]` tag resolving to a `NET.md` row?
2. **Earns** — does it contribute *unique* protection? The check is concrete, not aesthetic: would deleting this test reduce what the net catches? If no mutation turns *only this test* red, the answer is no.

## Rent-defaulters, by name

| Defaulter | How you catch it |
|---|---|
| **Untagged** — no `[net: ...]` tag | grep test names/docstrings for the tag; absence is rent-defaulting by definition |
| **Duplicate** — two tests red for the same mutation, and no mutation reds this one alone | rerun the mutation from the behavior's evidence file (or a fresh one); record which tests fire |
| **Tautology** — cannot go red | mutate the guarded line; the test stays green → it guards nothing, provable on record |
| **Framework-test** — asserts the library, not the project | the assertion would pass in an empty project using the same library |
| **Over-budget** — cost with a cheaper equivalent | the same behavior is provable at a lower level (an end-to-end walk duplicating a unit check) at a fraction of the wall-clock |

## The disposition ladder

Work down; stop at the first rung that fits. Every disposition is recorded.

1. **Delete** — the default for a defaulter. Update the affected `NET.md` row honestly in the same motion (`mapping-the-net`): if the behavior keeps other proven protection, prune the test pointer; if this was its only test, the row flips to `gap` (row lifecycle: `mapping-the-net`) or `R0(<reason>)` if non-protection is now the deliberate choice.
2. **Merge** — overlapping tests become one covering both checks; the merged test walks `proving-by-failure` like any new test.
3. **Tag-and-map** — the test protects a real behavior `NET.md` never mapped. That is a *new row*, not a deletion: enter it via `mapping-the-net` (as `gap` with the test pointer), add the `[net: ...]` tag, and the row goes `protected` only once the test has a red demo on record (`proving-by-failure`). An untagged protector is a mapping failure, not a freeloader.

**Never on the chopping block:** the sole protection of an R2+ behavior is not deleted for cost — it is merged or rewritten cheaper, or the cost is eaten. Consequence allocates effort, not runtime.

## Recording

Deletion is a first-class recorded outcome:

- One worklog `decision` line per disposition — test, defaulter type, rung taken, reason (`keeping-state`).
- The `NET.md` change lands in the SAME commit as the test deletion or merge (`landing-the-tests` atomicity: protection and its map never diverge in history).

## Stop conditions

- A deletion would leave an R2+ behavior unprotected with no recorded reason → stop; that is a rewrite or `backfilling-the-gap` decision, not a quiet eviction.
- Mutation evidence is ambiguous — you can't tell duplicate from complementary → keep both, worklog the ambiguity, let the next sweep's mutation audit settle it.
- The test belongs to an active main-ratchet task → off-limits until that task lands.

## Rationalization check

| Thought | Reality |
|---|---|
| "It's green and harmless, leave it" | Harmless tests cost wall-clock, and wall-clock is why suites stop being run. Rent or eviction. |
| "Deleting a test reduces coverage" | The net is measured in behaviors, not lines. A duplicate or tautology protected nothing; `NET.md` now says so honestly. |
| "This untagged test looks useful — keep it as-is" | Untagged is illegible to the next session and defaults again next census. Tag-and-map it or evict it. |
