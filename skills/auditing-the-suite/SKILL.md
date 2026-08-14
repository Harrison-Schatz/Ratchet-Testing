---
name: auditing-the-suite
description: Use when the sweep is due — every 5 harvests or a suite-runtime budget breach per HARVEST.md config, whichever comes first — typically queued by `harvesting-signals`. Also use when the user says "audit the suite", "is the net still real", or "why is the suite so slow", or when NET.md rows have gone unre-proven past a sweep cycle. The map is a claim until checked.
---

# Auditing the Suite

`NET.md` says which behaviors are protected. Code drifts, tests rot, and past sessions lie accidentally — so the sweep re-proves a sample of those claims the only way a test is ever proven: by watching it go red. A row that stays green while its behavior is broken is not protection; it is a liar wearing a green badge.

**Prevents:** failure mode #12 (stale trust in the map) and #7 (suite cost creep); its blocking rule is the enforcement arm of #4 (suite rot).

## When the sweep is due

Read the sweep-cadence config in `HARVEST.md` (owned by `harvesting-signals`): **every 5 harvests OR suite runtime over budget, whichever first** (provisional — amend in place with a logged reason). A due sweep queues behind pins, R3 gaps, hardening, and lower-class backfill — audits are last, preempted only by an explicit human request.

## Step 1 — Draw the sample, on record

| Class | Sampled |
|---|---|
| R3 | **every row, every sweep** — membership in the sample IS the R3 proof standard, no exceptions |
| R2 | one third of rows, rotating so every R2 row recurs within three sweeps (provisional — amend in place with a logged reason) |
| R1 | rarely — at most one row per sweep |
| R0 | never mutated — but re-read each `R0(<reason>)`: does the reason still hold? Stale → worklog `surprise`, queue re-sizing (`sizing-the-tests`) |

Write the sample drawn — the behavior IDs — into the worklog **before** running anything. A sample chosen after seeing results is not a sample.

## Step 2 — Re-prove each sampled row

1. Open `.ratchet-testing/evidence/<area>--<slug>.md` and find the last red demo: which break (revert / stash / mutate), on which line.
2. Re-apply that break — or a fresh mutation of the guarded line if the code has drifted past the old one (record which you used).
3. Run the row's tests. **Expected: red.**
4. Restore; prove restoration with `git status`/`git diff` EMPTY; re-run: green.
5. Verdict:
   - **Went red** → the row holds. Append a dated mutation-audit entry to the evidence file (`proving-by-failure` owns the format).
   - **Stayed green** → the row is a LIAR. Flip its status to `gap` (`mapping-the-net`), queue backfill (`backfilling-the-gap`), worklog `surprise`. Do not fix the test inside the sweep — the sweep measures; the queue repairs.

Two tests going red for the same mutation is duplicate protection — note it for the `charging-the-rent` census; do not delete mid-sweep.

## Step 3 — Check the runtime budget

Compare the suite's wall-clock time from this sweep's full run against the budget in `HARVEST.md` config. Over budget → queue a `charging-the-rent` census. A net too slow to run protects nothing — cost creep is how nets end up undeployed.

## Step 4 — THE BLOCKING RULE

Read `FLAKES.md`. Any quarantine past its budget end with disposition still OPEN means **the sweep cannot be declared clean.** The report's final line is `BLOCKED on flake dispositions: <entries>` — full stop, no "otherwise healthy" softening. Rot is made structurally loud, not quietly cumulative.

## Step 5 — Record

- Dated mutation-audit entries in each sampled behavior's evidence file (done per row in Step 2).
- One worklog `evidence` line: sample drawn, liars found, runtime vs budget, verdict — `clean` or `BLOCKED on flake dispositions`.
- Every NET.md flip lands atomically with the change that caused it (`landing-the-tests`).

## Stop conditions

- A sampled test shows different verdicts on the same code mid-sweep → `quarantining-the-flake` immediately; the sweep continues around it.
- A mutation cannot be applied because the behavior no longer has a seam → `requesting-the-seam`; flip the row to `gap`.
- Liars clustering on one fixture or mock pattern → a systemic cause, not row-by-row rot. Stop flipping rows; worklog `surprise` and present the pattern to the human.

## Rationalization check

| Thought | Reality |
|---|---|
| "The suite is green, the sweep can wait" | Green is exactly what a dead net shows. The sweep exists because green proves nothing. |
| "Skip the R3 rows — they passed last sweep" | Every-sweep membership IS the R3 proof standard. Skipping silently downgrades the class. |
| "One overdue flake, but everything else is healthy — call it clean-ish" | The blocking rule has no clean-ish. BLOCKED, full stop, is the report. |
| "This liar test just needs a quick fix while I'm here" | The sweep measures; the queue repairs. Fixing mid-sweep buries the finding the sweep exists to surface. |
