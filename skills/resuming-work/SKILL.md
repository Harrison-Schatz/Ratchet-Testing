---
name: resuming-work
description: Use at the start of any session where .ratchet-testing/STATE.md's roster lists an active task, when the user says "continue the testing work", "pick up where we left off", or references testing tasks you have no memory of, or after a context compaction mid-task. Resume from disk — do not reconstruct from guesswork or restart tests that already exist.
---

# Resuming Work

A fresh session knows nothing; the previous one wrote everything needed into `.ratchet-testing/`. Resume = read, verify the claims against the repo, reconcile, continue. This is the parent's `resuming-work` pointed at this system's state — with one upgrade: here the map itself (NET.md) is among the claims to check.

**Prevents:** failure mode #12 (stale trust in the map — resume verifies the roster against the actual suite before continuing); also context loss between sessions, outside the numbered table.

## Step 1 — Read the record

1. `.ratchet-testing/STATE.md` roster → find the task's row (the user names it, or there is exactly one active row).
2. `state/<task-id>.md` — class, type, phase, step, NEXT ACTION, blocked-on, pointers.
3. `worklog/<task-id>.md` — decisions, surprises, escalations, and the `proof` entries' evidence pointers.
4. The NET.md rows and `evidence/<area>--<slug>.md` files the task touched (pointers in the state file).

## Step 2 — Verify before continuing

The state file is a claim. The map is a claim until checked — **your past self lies accidentally.**

```
git status                  # only test paths and .ratchet-testing/ touched? matches the state file?
git log --oneline -10       # test-only commits matching the recorded step?
<project test command>      # a claim of green is a claim — run the suite
```

Then spot-check the NET.md rows the task touched: every `protected` row's test pointer must resolve to a real test carrying its `[net: <behavior-id>]` tag, and its proof pointer to a dated entry in `evidence/`.

Before touching any test file, re-run the deconfliction check — the main ratchet may have claimed test paths since your session died: read `.ratchet/STATE.md`'s roster (`harvesting-signals` owns this check; `.ratchet/` stays read-only).

| Finding | Meaning | Action |
|---|---|---|
| Repo matches the state file exactly | clean death at a boundary | execute the NEXT ACTION sentence |
| Test changes exist beyond the last recorded step | died after working, before recording | keep the work — but an unrecorded test has NO red demo on record; it re-enters at `proving-by-failure`, never at "done" |
| State or NET.md claims more than the repo shows | recorded optimistically | **trust the repo over the state file**: correct the state file and roster; a `protected` row missing its test or proof pointer flips back to `gap` via `mapping-the-net`; worklog `surprise` if material |
| Suite red where the last entry claimed green | drift, regression, or flake | rerun once: same code, different verdicts = flake → `quarantining-the-flake`; consistently red = the net caught something real — worklog `surprise`, treat it as harvested work |

## Step 3 — Reconcile and continue

1. Correct the state file and roster row to verified reality; worklog `surprise` if the drift was material (a one-line `decision` if trivial).
2. Tell the user in 2–4 sentences: task, step N of M, anything reconciled, what happens next. Do not re-litigate decisions already in the worklog — the record is the memory.
3. Execute the NEXT ACTION sentence. **Never restart work that exists on disk** — a test with a dated evidence/ entry keeps its proof; rewriting it from scratch discards a witnessed failure for nothing.

## Stop conditions

- No `.ratchet-testing/` exists but the user says "continue": say so plainly. If they mean dev work, that is the parent's `resuming-work` against `.ratchet/` — not this skill. Do not fabricate continuity.
- State and repo disagree beyond reconciliation: worklog `blocked`, present what you verified, ask the human. **Do not guess.**

## Rationalization check

| Thought | Reality |
|---|---|
| "The worklog says the red demo happened" | A `proof` entry is a pointer, not proof. No dated red output in `evidence/` → the demo re-runs. |
| "The suite was green last session — skip the run" | A claim of green is a claim. Two minutes buys a verified baseline. |
| "These half-done tests are messy; cleaner to start over" | Restarting discards proven clicks. Keep what has evidence; re-prove what doesn't. |
