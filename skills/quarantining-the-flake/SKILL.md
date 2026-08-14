---
name: quarantining-the-flake
description: Use the MOMENT any test shows different verdicts on the same code — mid-suite-run, mid-sweep, mid-resume, mid-anything — BEFORE trusting or dismissing either verdict. Also use when the user says "that test is flaky", when a rerun "fixed" a failure, or when a quarantine's disposition budget comes due. A flake is corrupted evidence — worse than no evidence.
---

# Quarantining the Flake

A flake is not a broken test — it is corrupted evidence. It degrades the authority of every green run and trains humans and agents to ignore red. Four rules: quarantine on first confirmation, never let the map overstate the net, dispose within budget, and let overdue rot block the sweep.

**Prevents:** failure mode #4 (suite rot / alarm fatigue).

## Step 1 — Confirm before quarantining

Confirmed flake = **same test, same code, different verdicts.** Rerun the test on the identical commit with an unchanged working tree:

- Different verdicts across runs → confirmed. Proceed to Step 2.
- Consistently red → not a flake — the net may have caught a real break. Report it; **never quarantine a genuine regression to make the suite green.**
- One odd failure that won't reproduce → a suspicion, not a confirmation. Worklog a `surprise` entry and watch.

## Step 2 — Quarantine on first confirmation, immediately

Append an entry to `.ratchet-testing/FLAKES.md` (this skill owns the format):

```
## <test path::name> — quarantined YYYY-MM-DD
- behavior: <area>/<slug>[#<check-slug>]
- verdict pattern: <e.g. pass, pass, FAIL, pass — 4 runs @ commit abc1234>
- repro context: <runner, parallelism, seed, machine, timing — anything correlated>
- clock start: YYYY-MM-DD, session <n>
- budget end: 3 testing sessions or next sweep, whichever first (provisional — amend in place with a logged reason)
- disposition: OPEN
```

Three effects, same moment:

1. **Authority stripped.** The test's verdict is excluded from the gate-relevant set — it can no longer pass or fail anything. It keeps executing for data; the verdict pattern is diagnostic.
2. **The map never overstates the net.** Flip the behavior's NET.md row to `suspended` in the same commit as the FLAKES.md entry (row lifecycle: `mapping-the-net`).
3. Worklog `quarantine` entry: test, behavior ID, pointer to the FLAKES.md entry.

## Step 3 — Disposition within budget

Budget: **3 testing sessions or the next sweep, whichever first** (provisional — amend in place with a logged reason). Exactly one of three outcomes, recorded on the entry's `disposition:` line:

| Disposition | Mechanics |
|---|---|
| **Deflake** | Root-cause the nondeterminism — the parent's `debugging-to-root-cause` applies unchanged (flakiness is data: timing, ordering, shared state). Fix the TEST, prove stability by N consecutive clean runs under repetition/stress, record N in the entry. Then restore authority and flip the NET.md row back to `protected`. |
| **Rewrite** | The test's approach was unsound. The replacement walks `proving-by-failure` like any new test — red demo on record before it earns a NET.md pointer. Delete the flaky original. |
| **Delete** | Remove the test and record the honest resulting state in NET.md: `gap` (queues backfill at R2+) or `R0(<reason>)`. Deletion with an honest map is a first-class outcome; a lying green test was worth less than either. |

Close with a worklog `disposition` entry naming the outcome and why. The FLAKES.md entry itself stays — the register is append-only, and patterns across old flakes are how systemic causes get spotted.

## Overdue blocks the sweep

Overdue with disposition OPEN blocks the sweep (`auditing-the-suite` Step 4). Blocked-and-loud is legal; silent-and-overdue is not.

## Stop conditions

- The nondeterminism traces to production source (a real race, not a test artifact) → you cannot fix it; record it as an issue in `.ratchet-testing/issues/` for the human or main ratchet, keep the test quarantined, NET.md row stays `suspended`.
- Deconfliction check (`harvesting-signals`) before any test-file write. The flaky test is off-limits by that check → quarantine and suspend as usual (FLAKES.md and NET.md are yours), but do not edit the test file until the owning task lands.
- No seam to rewrite at → `requesting-the-seam`; disposition becomes Delete-with-`gap` plus a pointer to the request.

## Rationalization check

| Thought | Reality |
|---|---|
| "It passed on rerun, we're fine" | Two verdicts on one commit IS the failure. The rerun confirmed the flake, not the pass. |
| "It only flakes in CI — quarantine can wait" | Every gate it votes in is taxed already. Authority is stripped on confirmation, not on convenience. |
| "Three sessions is too tight for this one" | The budget line is amendable in place — with a logged reason. Silent overrun is the one option that doesn't exist. |
