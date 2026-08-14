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
4. Harvest: invoke `harvesting-signals` — cadence: every session start (provisional — amend in place with a logged reason). It diffs `.ratchet/` against the `HARVEST.md` watermark and queues tasks by rule. No `.ratchet/` in the repo → harvest is a no-op; the queue comes from `NET.md` gaps and human requests only.

## Step 1 — Route the signal

| Signal | Invoke | Task type |
|---|---|---|
| Tier 2/3 main-ratchet task lands (`.ratchet/STATE.md` roster, worklogs) | `hardening-the-evidence` | harden |
| Root-caused fix in `.ratchet/worklog/` debugging entries | `pinning-the-bug` | pin |
| New rule in `.ratchet/LESSONS.md` | `backfilling-the-gap` | backfill |
| Declined/deferred review finding (`.ratchet/review/`, `.ratchet/` issues) | `backfilling-the-gap` | backfill |
| Brief acceptance checks (`.ratchet/briefs/`) name unmapped behaviors | `mapping-the-net` → `backfilling-the-gap` | backfill |
| Gap at R2+ discovered in `NET.md` | `backfilling-the-gap` | backfill |
| Brief/plan names legacy behavior; R2+ behavior with no tests and no spec of record | `characterizing-the-behavior` | characterize |
| Scheduled sweep due (`HARVEST.md` config) | `auditing-the-suite` | — |
| Suite runtime over budget | `charging-the-rent` | — |
| Flake observed — mid-ANYTHING: same test, same code, different verdicts | `quarantining-the-flake`, immediately | — |
| Behavior untestable without a production-source change (no seam) | `requesting-the-seam` | — |
| "Is it done?", "the test works", ANY done-claim about a test | `proving-by-failure` | — |
| Active roster row, "continue", compaction mid-task | `resuming-work` | — |
| Explicit human request | as specified | any |

When multiple tasks are queued, the order is fixed: **pins → R3 gaps → hardening → lower-class backfill → audits.** An explicit human request preempts all.

## Step 2 — Walk the spine

```
ORIENT (+ HARVEST) → SIZE (against RISKS.md) → BUILD (battery-scaled) → PROVE → RECORD
```

- **ORIENT + HARVEST** — Step 0 above.
- **SIZE** — `sizing-the-tests`: mechanical lookup of the behavior(s) against `RISKS.md`; a task touching multiple behaviors inherits the highest class; one worklog `sizing` line. Criteria don't cover the behavior → back to `defining-the-risks`, never an improvised class.
- **BUILD** — the risk class sets the battery; the task type routes: harden → `hardening-the-evidence`, pin → `pinning-the-bug`, backfill → `backfilling-the-gap`, characterize → `characterizing-the-behavior`.
- **PROVE** — the gate: `proving-by-failure`. Every new or promoted test has a witnessed red on record in `evidence/`, the battery for the class is complete, and the suite is green after restoration. A pass is not proof. No red on record, no "done" — structurally blocked.
- **RECORD** — `landing-the-tests`: the test change, the `NET.md` row, and the evidence file land in the SAME commit; roster row cleared; worklog `done`. State is updated via `keeping-state` at every phase boundary, not at session end.

## The three write zones

| Zone | Access |
|---|---|
| Production source, `.ratchet/` | **read-only, always** |
| Project test paths | read-write |
| `.ratchet-testing/` | read-write (its own state) |

Before ANY test-file write, run the deconfliction check (`harvesting-signals`): test files owned by an active main-ratchet task in `.ratchet/STATE.md`'s roster are off-limits until that task lands.

## The three structural rules

1. **No `NET.md` row flips to `protected` without a proof pointer into `evidence/`.** The red demo exists first (`proving-by-failure`) or the status doesn't change.
2. **State is updated at every phase boundary** (`keeping-state`). A session that dies right now must be resumable from `.ratchet-testing/` alone.
3. **Never write production source or `.ratchet/`.** No seam → `requesting-the-seam`, never a smuggled edit. Worst case, this system breaks a test — that invariant is what buys gate-free task flow.

## If you're tempted to skip routing

The signal→skill table above is the whole contract, and it is what this system is audited against: every signal in the README must land on a predictable row. If a signal arrives and no row fits, that is a spec bug to record — not a license to improvise. Routing costs one table lookup; skipping it costs the predictability everything downstream depends on.

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
