---
name: pinning-the-bug
description: Use when harvest finds a debugging entry in .ratchet/worklog/ with a named root cause and a fix commit — every root-caused fix gets a pin, no exceptions. Also use when the user says "make sure this bug never comes back", or when an audit or resume reveals a fixed bug with no regression test guarding it. The regression window is open until the pin lands.
---

# Pinning the Bug

A fixed bug with no pin is a bug on layaway: nothing stops the next change from reintroducing it, and next time there is no fresh debugging session pointing at the cause. The pin is a test that **fails on the pre-fix code** — proven the only way a test can be, by watching it.

**Prevents:** failure mode #6 (regression without a pin — bug fixed, nothing stops it from returning).

## Step 1 — Intake

1. Input from `harvesting-signals`: a `.ratchet/worklog/` debugging entry — the parent's `debugging-to-root-cause` leaves them — naming a root cause and a fix commit. Both are required; a symptom description without a named cause pins nothing.
2. **The regression-window rule: pins outrank everything.** Queue priority is pins → R3 gaps → hardening → lower-class backfill → audits; a pin preempts whatever non-pin task would otherwise start. The window between fix and pin is the exposure.
3. Deconfliction (`harvesting-signals`): if the fix's test files belong to a still-active main-ratchet task, the pin waits for that land.
4. Size via `sizing-the-tests`: the behavior the bug broke, classified against RISKS.md. One worklog `sizing` line.

## Step 2 — Write the pin

1. Name the violated behavior as a check sub-entry on its capability row: `<area>/<slug>#pin-<bug-slug>` (e.g. `auth/valid-login#pin-expired-token-accepted`). Row missing from NET.md → `mapping-the-net` adds it first.
2. Write the test asserting the post-fix behavior — at a seam, against the interface, per the same coupling standard `hardening-the-evidence` applies. Tag it `[net: <area>/<slug>#pin-<bug-slug>]`.
3. Test paths only. The fix itself is production source: read it, never touch it.

## Step 3 — Prove against the pre-fix code

This IS `proving-by-failure` applied — that skill owns the mechanics and the evidence-entry format. The pin's specific shape:

1. Temporarily remove the fix: `git revert -n <fix-commit>`, or stash-apply a saved breaking patch of the fix hunk. Record which.
2. Run the pin → capture the verbatim RED output. **A pin that passes on pre-fix code is not a pin** — back to Step 2.
3. Restore; prove restoration with `git status`/`git diff` EMPTY.
4. Run the pin → green. Full suite → green.
5. Both outputs, dated, appended to `evidence/<area>--<slug>.md`. Worklog `proof` line pointing at the file — never pasting it.

## Step 4 — Land

`landing-the-tests`: pin, NET.md row (status `protected`, proof pointer), and evidence file in the same commit, message citing the behavior ID and task-id. Worklog `done`, roster row cleared. The regression window is now closed — say so in the worklog.

## Stop conditions

- The root cause has no seam to test at — hardwired dependency, no injection point, global state → `requesting-the-seam`. NET.md records the behavior as `gap` with a pointer to the request, and **the window stays open**: recorded, visible, re-checked by harvest until the seam lands. Never smuggle a seam.
- The fix commit will not revert cleanly and no breaking patch can be constructed → mutate the fixed line instead (`proving-by-failure`'s third mechanism). If even that cannot produce red, you do not understand the fix — reread the debugging worklog before writing more test.
- The worklog names a symptom but no root cause → nothing to pin yet; root-causing is the main ratchet's work, not yours.

## Rationalization check

| Thought | Reality |
|---|---|
| "The fix already has a test from the dev task" | Did it go red on the pre-fix code, on record? If not, it is an evidence-test that happened to pass. Prove it red or write the pin. |
| "This bug was a one-off, nobody would reintroduce it" | It got in once past everyone. The pin costs one test; the recurrence costs a second debugging session with none of today's context. |
| "The pin passes on current code — good enough" | A pass proves nothing. Red on pre-fix code is the entire point; without it you may have pinned nothing. |
| "This R3 gap is scarier, the pin can wait" | The gap has existed all along; the window opened this week where change is known to be live. Pins first, by rule. |
