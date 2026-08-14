---
name: backfilling-the-gap
description: Use when a queued backfill task is claimed — a NET.md row sitting at `gap` at R2+, a `suspended` row whose flake budget expired, a LESSONS.md invariant worth encoding, a declined review finding that names a behavior, or brief acceptance checks for behaviors entering the map. Also use when the human asks to "cover X" or "protect Y". Closes recorded gaps in priority order; a row flips to protected only behind a witnessed failure.
---

# Backfilling the Gap

A gap is honesty, not safety: `NET.md` admits the behavior is unprotected, and admission is only step one. Backfilling converts recorded gaps into protected rows — battery-scaled by risk class, each test proven red before the map is allowed to claim it.

**Prevents:** failure mode #9 (unknown blind spots).

## Step 1 — Know your gap's source

Backfill tasks arrive through `harvesting-signals`' queue (gaps and expired suspensions at R2+, LESSONS invariants, declined findings, harvested acceptance checks). This skill closes them, it does not discover them.

## Step 2 — Take them in order

Take gaps in queue order (queue rules: `harvesting-signals`). **Within a class, order by user impact** — rank against the project examples in RISKS.md: the behavior whose failure looks most like the class's worst example goes first.

## Step 3 — Size, then build the battery

1. Size via `sizing-the-tests`. One worklog `sizing` line.
2. Build the battery the class demands (table in `sizing-the-tests`). An R2+ legacy behavior with no spec of record gets its pin via `characterizing-the-behavior` first. Precedence: spec of record present → `backfilling-the-gap`; absent (the behavior exists in code, its intent never stated) → `characterizing-the-behavior` first. Brief acceptance checks are a spec of record.
3. Deconfliction check (`harvesting-signals`) before any test-file write.
4. Tag every test `[net: <behavior-id>]`; distinct checks get sub-entries — `auth/valid-login#expired-token`.

## Step 4 — Prove every test

Every new test walks `proving-by-failure`. A backfill test that has never been red is the gap wearing a costume.

## Step 5 — Flip the row, atomically

`NET.md` flips `gap` → `protected` **only with a proof pointer into evidence/** — never on a green run alone. The row change, the evidence file, and the tests land in the same commit (`landing-the-tests`). Worklog `done`, roster row cleared.

## Stop conditions

- RISKS.md is missing, or its criteria don't cover this behavior → `defining-the-risks` amendment; never an improvised class.
- No seam — the behavior can't be tested without a production-source change → `requesting-the-seam`; the row stays `gap` with a pointer to the request.
- The gap's test paths belong to an active main-ratchet task → skip it, take the next queued item, re-check at next harvest.
- Mid-build evidence the class is wrong → stop, worklog `escalation`, re-run `sizing-the-tests`; keep surviving work.

## Rationalization check

| Thought | Reality |
|---|---|
| "This gap has been open for months without incident" | Open-without-incident is survivorship, not safety. The class sets the priority, not the age. |
| "A green test closes the gap faster than the red-demo ritual" | A green-only test may be a tautology — the exact thing that reopens the gap invisibly. Failure mode #1 is why the demo exists. |
| "I'll flip the row now and add the proof pointer later" | A `protected` row without proof is exactly the lie `auditing-the-suite` exists to catch. Flip and proof are one commit. |
