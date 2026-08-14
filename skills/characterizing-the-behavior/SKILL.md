---
name: characterizing-the-behavior
description: Use when harvest surfaces a brief or plan naming legacy behavior someone is about to change, or when an R2+ behavior in NET.md has no tests and no spec of record. Also use to supply the characterization pin an R3 battery requires. Pins what the code does TODAY — bugs included — at a seam, test-writes only, so tomorrow's change can't silently rewrite it.
---

# Characterizing the Behavior

You cannot protect a behavior nobody has written down; a characterization test writes it down as executable fact. It asserts what the code *actually does right now* — not what it should do — so every future diff in behavior is one somebody chose. This is the parent's `characterizing-legacy-code` discipline, re-scoped to this system's boundary: production source stays read-only, always.

**Prevents:** failure mode #9 (unknown blind spots) on legacy behavior — and, beyond the numbered table, regressions in behavior nobody specified but everybody depends on.

## Step 1 — Trigger and mapping

Two triggers, both arriving via `harvesting-signals`:

1. A `.ratchet/briefs/` entry or plan names legacy behavior a dev task will change — characterize BEFORE that change lands, so the diff has a baseline.
2. An R2+ behavior has no tests and no spec of record — including the characterization pin every R3 battery requires (`sizing-the-tests`).

If the behavior has no `NET.md` row yet, it enters first via `mapping-the-net` — as `gap`, never silently. Before any test-file write, run the deconfliction check (`harvesting-signals`): test files owned by an active main-ratchet task are off-limits until it lands.

## Step 2 — Reach the behavior at a seam

Test-writes only. In order of cheapness: call the behavior directly from a test; call it with light scaffolding (temp dir, in-memory DB, fake clock, fixed seed). What you may NOT do is create a seam — that is a production-source edit. If no seam exists → `requesting-the-seam`; the row stays `gap` with a pointer to the request, and this task parks.

## Step 3 — Capture the golden master

1. Build an input grid per input axis: typical, boundary (empty / zero / null / max), and at least one garbage input — "what does it currently do with garbage?" is a behavior too.
2. Run the CURRENT code over the grid and capture the actual output. Don't assert your model of the behavior — assert the captured output. (Cheap trick from the parent: assert a wrong-but-plausible value first; the failure message tells you the real one.)
3. The test asserts current output verbatim, **bugs included**. Golden-master files are legitimate for large outputs (reports, serialized structures): capture once, diff against the master thereafter.
4. Nondeterminism (time, random, ordering): control it at the seam — fake clock, fixed seed — or assert invariant properties instead of exact values. Never pin output you can't reproduce.
5. Tag `[net: <behavior-id>]`; name tests for what they record: `renders 2-digit totals unpadded (current behavior)`. Distinct findings get check sub-entries: `billing/invoice-render#garbage-input`.

## Step 4 — Surprises become issues, never fixes

Found a bug, an oddity, a security smell? **Pin it as-is** and record it as a candidate issue — a dated file in `.ratchet-testing/issues/` (the directory `requesting-the-seam` owns; follow its file conventions), plus a worklog `surprise` line. Never fix in flight: the write zone forbids the production edit, and a silent fix changes the very behavior you are pinning.

## Step 5 — Prove the pins

Characterization tests are green against unchanged code by construction — so green proves nothing here either. `proving-by-failure` applies unchanged: temporarily mutate the captured behavior path, watch the test go red, restore (git status/diff empty), watch green, append both outputs to `.ratchet-testing/evidence/<area>--<slug>.md`. A pin that stays green under mutation pins nothing.

## Step 6 — Record

`NET.md` row → `protected` with the proof pointer; row, evidence, and tests land in one commit (`landing-the-tests`). Worklog `done`; roster row cleared.

## Stop conditions

- No seam, and light scaffolding can't reach the behavior → `requesting-the-seam`; do not force a mock jungle that tests the mocks.
- Behavior is environment-dependent and unreproducible here → don't pin what you can't observe; worklog `blocked`, leave the row `gap` with the reason.
- The behavior's test paths belong to an active main-ratchet task → off-limits until it lands; take the next queued item.
- The input grid keeps growing past the class's battery → re-run `sizing-the-tests`; the row may need splitting into capability row + check sub-entries.

## Rationalization check

| Thought | Reality |
|---|---|
| "This output is obviously a bug — one line fixes it" | The write zone forbids the edit, and fixing mid-pin changes the thing being measured. Pin it, file the issue, let the main ratchet fix it against your net. |
| "I can read the code and write the expected values" | Reading gives intent; years of patches give behavior. Run the code; assert what it returned. |
| "The tests are green, so the behavior is pinned" | Green by construction. Only the mutation-red in evidence/ proves the pin holds weight. |
| "Characterization tests aren't real tests" | They are the only spec this behavior has — and after the next change, they are its regression net. |
