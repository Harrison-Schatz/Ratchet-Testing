---
name: hardening-the-evidence
description: Use when harvest queues a reinforcement pass after a main-ratchet Tier 2/3 task lands — the landed change's evidence-tests exist, but nothing says they will survive a refactor. Also use when a dev-suite test looks load-bearing but asserts internals, or when deciding whether a task-scoped test deserves a NET.md row. Promotion is earned by checklist, never assumed.
---

# Hardening the Evidence

The main ratchet writes tests to pass a done-gate; they prove today's change worked, not that tomorrow's breakage will be caught. This skill runs each landed evidence-test through the coupling checklist and gives it exactly one disposition — promote, rewrite at the seam, or leave dev-scoped — recorded either way.

**Prevents:** failure mode #5 (implementation-coupled tests — false alarms on refactor erode trust in the net).

## Step 1 — Claim and scope

1. Input arrives from `harvesting-signals`: a Tier 2/3 land observed in `.ratchet/`, queued as a `harden` task.
2. Deconfliction check (`harvesting-signals`) before any test-file write.
3. Size via `sizing-the-tests`. One worklog `sizing` line.
4. List the landed change's evidence-tests from its commit and worklog pointers. **Each test gets its own verdict** — never harden "the batch."

## Step 2 — The coupling checklist

Run every candidate through all five. Each check is a yes/no you could defend from the test file alone:

| Check | The question |
|---|---|
| Behavior at a seam | Does it assert observable behavior at a seam, not internals? Would a pure refactor — same behavior, different code — break it? If yes, it is coupled. |
| Mock placement | Are mocks only at architectural boundaries (network, DB, clock, filesystem)? Is anything mocked that lives inside the unit under test? If yes, it tests the mock. |
| Name | Does the name state the behavior ("expired token is rejected"), not the implementation ("calls validateJWT")? |
| Tag | Does it carry a `[net: <behavior-id>]` tag resolving to a NET.md row? |
| Interface-only | Does it survive running against the public interface only — no reaching into private state to arrange or assert? |

## Step 3 — Disposition, one per test

- **Promote** — passes all five (adding the tag if that is the only miss): the test enters the net. NET.md row updated via `mapping-the-net`, and the promotion walks `proving-by-failure` — a promoted test with no red demo on record is not promoted.
- **Rewrite at the seam** — the behavior deserves protection but the test is coupled: write a new test at the seam, and the new test walks `proving-by-failure` like any other.
- **Leave dev-scoped** — recorded, not silent: the test keeps passing in the dev suite but claims no NET.md protection. Worklog `decision` line naming the test and why. If the behavior it half-covered is R2+ and unprotected, that is a gap (row lifecycle: `mapping-the-net`).

## Step 4 — Land

`landing-the-tests` owns the commit: test change, NET.md row, and evidence file land atomically. Worklog `done`, roster row cleared.

## Stop conditions

- The behavior cannot be asserted at any seam — everything observable requires internals → `requesting-the-seam`; NET.md records the gap. Never promote a coupled test as "better than nothing."
- The test files belong to an active main-ratchet task → wait for its land; re-queue the harden.
- The landed change's behaviors are not in NET.md at all → `mapping-the-net` adds the rows first (they enter as gap), then harden.

## Rationalization check

| Thought | Reality |
|---|---|
| "It passed the dev gate, it's already a good test" | It proved the change worked once. The checklist asks whether it catches the break in two years under a different author. Different question. |
| "It's coupled, but coverage is coverage" | A test that fails on every refactor trains everyone to ignore red — failure mode #5, the exact thing this pass keeps out of the net. |
| "I'll promote it now and prove it red later" | No red on record, no net membership. `proving-by-failure` is part of promotion, not an afterthought. |
| "Leaving it dev-scoped feels like giving up" | Recorded non-promotion is honest; NET.md shows the gap. Silent promotion of a coupled test is the map lying. |
