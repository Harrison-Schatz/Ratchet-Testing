---
name: using-ratchet-testing
description: Load at the start of EVERY testing session and before acting on ANY signal that could add, change, quarantine, or delete a test — including work that looks trivial ("just add a quick test", "rerun that flaky one"). Also load whenever a .ratchet-testing/ directory exists in the repo, whenever you are unsure which Ratchet-Testing skill applies, or whenever you are about to touch a test file without a sized task behind it.
---

# Using Ratchet-Testing

Ratchet-Testing is the sub-ratchet that builds and maintains the test suite as a durable safety net. The main ratchet delivers changes; this system delivers and maintains the net — it hardens, pins, backfills, audits, and prunes, and it never writes production source. This skill routes every signal to the skill that handles it.

**Prevents:** the meta-failure of skills never firing — none of failure modes #1–#12 is defeated if its skill doesn't fire on its signal. If a routing decision isn't predictable from this file plus the README, that's a bug — file it.

## Step 0 — Orient (always, before anything else)

1. Read `.ratchet-testing/STATE.md` — the roster of ACTIVE testing tasks.
   - **Lists an active task** → this is a resume. Invoke `resuming-work` with that task-id. STOP here; that skill takes over.
   - **Empty or missing** → continue.
2. Check for `.ratchet-testing/RISKS.md`. **If it doesn't exist or is unapproved, `defining-the-risks` is the only legal first task** — nothing above R0 can be classified against an unapproved risk model, and everything downstream sizes against it. STOP routing until it exists.
3. Read `.ratchet-testing/NET.md` — what is `protected`, `suspended`, `gap`, or `R0(<reason>)` right now. The map is a claim until checked; audits and resumes verify it.
4. Harvest: invoke `harvesting-signals` — cadence: per `HARVEST.md` Config (`harvesting-signals`). It diffs `.ratchet/` against the `HARVEST.md` watermark and queues tasks by rule. No `.ratchet/` in the repo → harvest is a no-op; the queue comes from `NET.md` gaps and human requests only.

## Step 1 — Route the signal

| Signal | Invoke | Task type |
|---|---|---|
| Tier 2/3 main-ratchet task lands (`.ratchet/STATE.md` roster, worklogs) | `hardening-the-evidence` | harden |
| Root-caused fix in `.ratchet/worklog/` debugging entries | `pinning-the-bug` | pin |
| New rule in `.ratchet/LESSONS.md` | `backfilling-the-gap` | backfill |
| Declined/deferred review finding | `mapping-the-net` records the behavior's NET.md row (gap or R0-with-reason, pointing at the finding — the risk-register entry), then `backfilling-the-gap` for any resulting R2+ gap | backfill |
| Brief acceptance checks appear (`.ratchet/briefs/`) | `mapping-the-net`, then `backfilling-the-gap` for any resulting R2+ gap | backfill |
| Gap at R2+ discovered in `NET.md` | `backfilling-the-gap` | backfill |
| Legacy behavior about to change, or R2+ gap with no spec of record | `characterizing-the-behavior` | characterize |
| Scheduled sweep due (`HARVEST.md` config) | `auditing-the-suite` | — |
| Suite runtime over budget | `auditing-the-suite` (sweep due now; the sweep queues `charging-the-rent` for the census) | — |
| Flake observed — mid-ANYTHING: same test, same code, different verdicts | `quarantining-the-flake`, immediately | — |
| Behavior untestable without a production-source change (no seam) | `requesting-the-seam` | — |
| "Is it done?", "the test works", ANY done-claim about a test | `proving-by-failure` | — |
| Active roster row, "continue", compaction mid-task | `resuming-work` | — |
| Explicit human request | as specified | any |

Precedence: spec of record present → `backfilling-the-gap`; absent (the behavior exists in code, its intent never stated) → `characterizing-the-behavior` first. Brief acceptance checks are a spec of record. (A spec of record is any stated intended behavior: brief acceptance checks, a LESSONS.md invariant, a RISKS.md example, or a declined finding's own text.)

When multiple tasks are queued, the order is fixed: **pins → R3 gaps → hardening → lower-class backfill → audits.** Characterize tasks take the rank of the gap they close: an R3 pre-change characterization ranks with R3 gaps; others rank with their class's backfill. An explicit human request preempts all; a signal with no fitting row is a spec bug to record, never a license to improvise.

## Step 2 — Walk the spine

```
ORIENT (+ HARVEST) → SIZE (against RISKS.md) → BUILD (battery-scaled) → PROVE → RECORD
```

- **ORIENT + HARVEST** — Step 0 above.
- **SIZE** — `sizing-the-tests`: mechanical lookup of the behavior(s) against `RISKS.md`; a task touching multiple behaviors inherits the highest class; one worklog `sizing` line. Criteria don't cover the behavior → back to `defining-the-risks`, never an improvised class.
- **BUILD** — the risk class sets the battery; the task type routes: harden → `hardening-the-evidence`, pin → `pinning-the-bug`, backfill → `backfilling-the-gap`, characterize → `characterizing-the-behavior`.
- **PROVE** — the gate: `proving-by-failure`. Every new or promoted test has a witnessed red on record in `evidence/`, the battery for the class is complete, and the suite is green after restoration. No red on record, no "done" — structurally blocked.
- **RECORD** — `landing-the-tests`: the test change, the `NET.md` row, and the evidence file land in the SAME commit; roster row cleared; worklog `done`. State is updated via `keeping-state` at every phase boundary, not at session end.

## The three write zones

| Zone | Access |
|---|---|
| Production source, `.ratchet/` | **read-only, always** |
| Project test paths | read-write |
| `.ratchet-testing/` | read-write (its own state) |

Deconfliction check (`harvesting-signals`) before any test-file write.

## The three structural rules

1. **No `NET.md` row flips to `protected` without a proof pointer into `evidence/`.** The red demo exists first (`proving-by-failure`) or the status doesn't change.
2. **State is updated at every phase boundary** (`keeping-state`). A session that dies right now must be resumable from `.ratchet-testing/` alone.
3. No seam → `requesting-the-seam`, never a smuggled edit (zones above).

## If you're tempted to skip routing

Routing costs one table lookup; skipping it costs the predictability everything downstream depends on.

## Stop conditions

- **No `RISKS.md`, or it's unapproved** → `defining-the-risks`. Nothing above R0 may be classified; no other task is legal first.
- **Roster lists an active task** → `resuming-work`. Do not start new work over it.
- **The write you're about to make lands in production source or `.ratchet/`** → stop. If a seam is missing, `requesting-the-seam`.
- **The test file belongs to an active main-ratchet task** → off-limits until that task lands.

## Rationalization check

| Thought | Reality |
|---|---|
| "This test is tiny — no need to size it." | Sizing is one lookup against `RISKS.md` and one worklog line. The judgment already happened, on record; you are only reading it. |
| "The new test passes, so it works." | A test that cannot fail also passes. Only a witnessed red (`proving-by-failure`) puts it in the net. |
| "I'll fix this production bug while I'm in here." | Production source is read-only, always. Record it in `.ratchet-testing/issues/` and leave the fix to the main ratchet. |
| "That flake will probably pass on rerun." | A confirmed flake is corrupted evidence — quarantine now (`quarantining-the-flake`), or every green run's authority decays. |
