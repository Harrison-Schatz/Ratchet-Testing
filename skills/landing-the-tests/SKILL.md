---
name: landing-the-tests
description: Use when proving-by-failure has passed and a testing task's work must become permanent — committing the tests, updating NET.md, clearing the roster row. Also use when deciding what rides in a testing commit, or when tempted to commit the tests now and update the map later. Nothing lands while its red demo is missing.
---

# Landing the Tests

Protection that exists only in a working tree protects nothing, and a map updated "later" is a map that lies in history. This skill is mechanics: one atomic commit that changes the net and its map together, then a clean record for whoever resumes next.

**Prevents:** failure mode #12 (stale trust in the map) at its source — protection and its map never diverge in history; plus protection-claimed-but-never-integrated, which sits outside the numbered table.

## Step 0 — Preconditions

- `proving-by-failure` passed: every new or promoted test in this task has a red demo in `evidence/`, and the full suite is green after restoration. **This skill never lands anything whose red demo is missing** — not "land now, prove next session." Missing red → STOP, back to the gate.
- `git status` shows changes ONLY under project test paths and `.ratchet-testing/`. Any production-source or `.ratchet/` diff means the gate's restore step failed — fix that before proceeding.

## Step 1 — The atomic commit

**The rule lives here:** the test change, the `NET.md` row change, and the `evidence/` file land in the SAME commit. Never split. A commit that flips a row to `protected` carries the proof pointer AND the pointed-at entry AND the test — the map is updated at the moment protection changes, not at the end of a session. A multi-behavior task landing in stages gives each commit its own `NET.md` delta.

State bookkeeping (worklog entry, roster row, `state/<task-id>.md`) rides in the same commit — nothing of the land is left for a "cleanup commit."

## Step 2 — Commit message convention

Testing commits are test-only and cite the behavior ID(s) plus the task-id:

```
test(net): pin expired-token rejection — auth/valid-login#expired-token

task: 2026-08-14-pin-expired-token
type: pin        # harden | pin | backfill | characterize
evidence: .ratchet-testing/evidence/auth--valid-login.md
```

## Step 3 — Verify, then land

Check the staged diff, not your memory:

- [ ] `git diff --cached --stat` — every path is a project test path or `.ratchet-testing/`. Anything else → unstage and abort.
- [ ] Every row flipping to `protected` has a proof pointer, and the pointed-at evidence entry is in this same staged set.
- [ ] Every durable test in the diff carries its `[net: <behavior-id>]` tag.
- [ ] Deconfliction re-check: no staged test file belongs to an active main-ratchet task per the `.ratchet/STATE.md` roster (the check is `harvesting-signals`'; re-run it here because land time is write time).
- [ ] Suite green on the exact tree being committed.

Then commit. Test-only commits usually land directly on the default branch — the structural invariant holds: worst case this commit breaks a test, never the shipped system. If the project's convention demands branches or PRs, follow it (determine the convention the way the parent's `landing-the-change` does). Never force-push; never rewrite history that contains evidence.

## Step 4 — Close the record

1. Worklog `done` entry: task-id, commit SHA, behavior IDs touched, evidence pointers (formats: `keeping-state`).
2. Clear the task's row from the `.ratchet-testing/STATE.md` roster; archive/delete `state/<task-id>.md`. The roster holds ACTIVE tasks only — a landed task's permanent record is its worklog.
3. Rows this land could not flip (a gap left open, a seam still pending) stay honest in `NET.md` — `mapping-the-net` owns the vocabulary; never mark `protected` on hope.

## Stop conditions

- Red demo missing for ANY test in the diff → back to `proving-by-failure`. No partial credit.
- Staged diff touches production source or `.ratchet/` → write-zone violation; restore, then re-verify from Step 0.
- A staged test file turns out to be owned by an active main-ratchet task → do not land it; worklog `blocked`, re-check at the next harvest.
- Suite red on the final tree → the land never saw a green gate; back through `proving-by-failure`.

## Rationalization check

| Thought | Reality |
|---|---|
| "Commit the tests now, update NET.md after" | That history contains a moment where the map lies — failure mode #12 by construction. Same commit. |
| "The red demo happened; the entry can wait" | An unrecorded demo is a claim. `evidence/` rides in the commit or the row doesn't flip. |
| "One tiny prod tweak rode along, it's harmless" | This system cannot break prod only because it never writes prod. Unstage it; if it's a seam you need, `requesting-the-seam`. |
| "Roster cleanup next session" | The next session resumes from the roster. A stale active row means re-verifying or restarting landed work. |
