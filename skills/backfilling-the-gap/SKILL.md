---
name: backfilling-the-gap
description: Use when a queued backfill task is claimed — a NET.md row sitting at `gap` at R2+, a `suspended` row whose flake budget expired, a LESSONS.md invariant worth encoding, a declined review finding that names a behavior, or brief acceptance checks for behaviors entering the map. Also use when the human asks to "cover X" or "protect Y". Closes recorded gaps in priority order; a row flips to protected only behind a witnessed failure.
---

# Backfilling the Gap

A gap is honesty, not safety: `NET.md` admits the behavior is unprotected, and admission is only step one. Backfilling converts recorded gaps into protected rows — battery-scaled by risk class, each test proven red before the map is allowed to claim it.

**Prevents:** failure mode #9 (unknown blind spots).

## Step 1 — Know your gap's source

Backfill tasks arrive through `harvesting-signals`' queue; this skill closes them, it does not discover them. Five sources feed it:

| Source | Signal |
|---|---|
| `NET.md` | row status `gap` at R2+ (auto-queued at harvest) |
| `NET.md` via `quarantining-the-flake` | row `suspended` past the flake disposition budget — 3 testing sessions or next sweep (provisional — amend in place with a logged reason) |
| `.ratchet/LESSONS.md` | a new rule stating an invariant → candidate test |
| `.ratchet/review/` and `.ratchet/` issues | a declined or deferred review finding that names a behavior |
| `.ratchet/briefs/` | acceptance checks for unmapped behaviors — the row enters `NET.md` first via `mapping-the-net`, as `gap`, never silently |

## Step 2 — Take them in order

Queue priority is fixed: pins → R3 gaps → hardening → lower-class backfill → audits. An explicit human request preempts all. **Within a class, order by user impact** — rank against the project examples in RISKS.md: the behavior whose failure looks most like the class's worst example goes first.

## Step 3 — Size, then build the battery

1. Run `sizing-the-tests`: mechanical lookup of the behavior(s) against RISKS.md — no judgment here; the judgment happened at `defining-the-risks`, on record. A multi-behavior task inherits the highest class. One worklog `sizing` line.
2. Build the battery the class demands (table in `sizing-the-tests`): R1 = happy-path smoke + one failure path; R2 = happy + failure + key edge cases; R3 = characterization pin + happy + failure + edges + invariant/property tests where feasible. An R3 legacy behavior with no spec of record gets its pin via `characterizing-the-behavior`.
3. Before ANY test-file write, run the deconfliction check (`harvesting-signals`): read `.ratchet/STATE.md`'s roster — test files owned by an active main-ratchet task are off-limits until it lands.
4. Tag every test `[net: <behavior-id>]`; distinct checks get sub-entries — `auth/valid-login#expired-token`.

## Step 4 — Prove every test

Every new test walks `proving-by-failure` — break the behavior (revert / stash / mutate), watch the red, restore, watch the green, append both outputs to `.ratchet-testing/evidence/<area>--<slug>.md`. A backfill test that has never been red is the gap wearing a costume.

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
| "It's only R1, zero tests is fine" | Less testing is never no testing above R0. R1 buys a smoke + one failure path; if it's truly not worth that, reclassify to `R0(<reason>)` on record. |
| "I'll flip the row now and add the proof pointer later" | A `protected` row without proof is exactly the lie `auditing-the-suite` exists to catch. Flip and proof are one commit. |
