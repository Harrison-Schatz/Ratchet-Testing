---
name: harvesting-signals
description: Use at every Ratchet-Testing session start — after orienting, before sizing or building anything — to pull new work by reading .ratchet/ against the watermark in HARVEST.md, and again whenever the queue runs empty mid-session. Also use BEFORE any write to a test file, because this skill owns the deconfliction check that keeps the two ratchets off the same files. Ratchet-Testing has no inbox: a signal that isn't harvested doesn't exist, and an unharvested root-caused fix is an open regression window.
---

# Harvesting Signals

Ratchet-Testing may only *read* `.ratchet/`, so all work is pulled, never pushed: one read pass over the main ratchet's artifacts, diffed against a watermark, converted into queued tasks by rule. This skill owns `HARVEST.md` — the watermark, the queue, the sweep-cadence config — and the deconfliction check.

**Prevents:** failure mode #6 (regression without a pin — the intake half) and #11 (the two ratchets colliding); also queue starvation, outside the table — the self-feeding queue is what makes this system session-independent.

## Step 1 — When

Harvest cadence: **every session start** (provisional — amend in place with a logged reason). One read pass is nearly free; staleness is the greater risk. Harvest again on demand when the queue empties mid-session. One exception: if the `.ratchet-testing/STATE.md` roster lists an active task, `resuming-work` takes precedence — harvest before claiming any *new* task, never instead of a resume.

## Step 2 — Read against the watermark

Open `.ratchet-testing/HARVEST.md`. For each source, read only what is newer than the recorded position, then advance the position. No file yet → create it from this template with the watermark empty; the first harvest reads everything, and that is fine.

```markdown
# HARVEST

## Watermark
| Source | Last-read commit | Date |
|---|---|---|
| .ratchet/STATE.md (roster) | <sha> | <YYYY-MM-DD> |
| .ratchet/worklog/ | <sha> | <YYYY-MM-DD> |
| .ratchet/LESSONS.md | <sha> | <YYYY-MM-DD> |
| .ratchet/review/ | <sha> | <YYYY-MM-DD> |
| .ratchet/briefs/ | <sha> | <YYYY-MM-DD> |

## Config
- Harvest cadence: every session start (provisional — amend in place with a logged reason)
- Sweep due: every 5 harvests OR suite runtime over budget, whichever first (provisional — amend in place with a logged reason)
- Suite runtime budget: <e.g. 90s wall-clock>
- Harvests since last sweep: <n — reset when the sweep runs>

## Queue (priority order — the roster holds only ACTIVE tasks)
| Task-id | Type | Behavior ID(s) | Class (per NET.md) | Source signal | Queued |
|---|---|---|---|---|---|
| 2026-08-14-pin-session-timeout | pin | auth/valid-login#pin-session-timeout | R3 | .ratchet/worklog/ 2026-08-12 root cause | 2026-08-14 |
```

## Step 3 — Convert signals to tasks

Each signal maps to a task type by rule — no judgment at intake:

| # | Signal (read from) | Generates | Routes to |
|---|---|---|---|
| 1 | Tier 2/3 task lands (`.ratchet/STATE.md` roster, worklogs) | reinforcement pass over the landed change's evidence-tests | `hardening-the-evidence` |
| 2 | Root-caused fix (`.ratchet/worklog/`, debugging entries) | regression pin | `pinning-the-bug` |
| 3 | New rule in `.ratchet/LESSONS.md` | candidate invariant to encode as a test | `backfilling-the-gap` |
| 4 | Declined/deferred review finding (`.ratchet/review/`, `.ratchet/` issues) | risk-register entry; backfill if it names a behavior | `backfilling-the-gap` |
| 5 | Brief acceptance checks (`.ratchet/briefs/`) | candidate durable tests for behaviors entering the map | `mapping-the-net` → `backfilling-the-gap` |
| 6 | Gap at R2+ discovered in `NET.md` (including suspensions) | backfill | `backfilling-the-gap` |
| 7 | Scheduled sweep due (counter or runtime budget, per Config) | suite health audit | `auditing-the-suite` |
| 8 | Human request | as specified | any |

Two supplementary passes ride along:

- A brief or plan naming legacy behavior about to change, where the behavior has no tests → queue `characterizing-the-behavior`.
- Re-check open seam requests in `.ratchet-testing/issues/` (`requesting-the-seam`): a landed seam queues the backfill or pin it was blocking.

A harvested behavior with no `NET.md` row enters the map as `gap` or `R0(<reason>)` via `mapping-the-net` — never silently.

## Step 4 — Queue and record

- **The queue lives in HARVEST.md** — one line per queued task, held in priority order: **pins → R3 gaps → hardening → lower-class backfill → audits.** An explicit human request preempts all.
- **The roster (`.ratchet-testing/STATE.md`) holds only ACTIVE tasks.** A task gets its roster row when claimed, not when queued (`keeping-state`). Formal classification happens at claim (`sizing-the-tests`); the queue's Class column is a copy of the `NET.md` row, kept only for ordering.
- Bump "Harvests since last sweep". At the Config threshold — or any time suite runtime is over budget — queue `auditing-the-suite`.
- One worklog `harvest` entry: sources advanced, signals found, tasks queued (`keeping-state` for where entries live).

## The deconfliction check — failure mode #11's answer

**Before ANY test-file write** — in every skill of this system, not just during harvest — read `.ratchet/STATE.md`'s roster:

- Test files under an active main-ratchet task's owned paths are **off-limits until that task lands.**
- Deconfliction is by reading, not locking — no messaging, no waiting protocol. Queue the affected task and take the next one in priority order.
- A roster row still present means active. Stalled, old, or suspicious is not landed; when ownership is ambiguous, treat the paths as owned.

The check lives here because harvest is the moment `.ratchet/` ownership is read; other skills point at this section rather than restating it.

## Graceful degradation

No `.ratchet/` directory → harvest is a no-op, not an error. The queue is fed from `NET.md` gaps at R2+ and human requests only. The main ratchet needs zero awareness that this system exists; absence in either direction breaks nothing.

## Stop conditions

- `RISKS.md` does not exist — stop queueing; `defining-the-risks` is the only legal first task (`using-ratchet-testing` Step 0).
- A signal names a behavior you cannot express as an `<area>/<slug>` ID — route through `mapping-the-net` before queueing; a task against an unmapped behavior cannot carry a `[net: <behavior-id>]` tag.
- `.ratchet/STATE.md` exists but is unreadable or contradicts the repo — do not guess ownership. Worklog `blocked`, ask the human.

## Rationalization check

| Thought | Reality |
|---|---|
| "The queue is already full; skip this harvest" | Pins outrank everything queued, and a regression window may have opened since the last read. One pass is nearly free. |
| "That main-ratchet task looks abandoned; its test files are fair game" | Off-limits until it *lands*. Stalled is not landed. Queue your task and take the next one. |
| "I'll put the new tasks straight on the roster" | The roster holds only ACTIVE tasks. A queue living in the roster makes every resume wade through dead rows. |
| "No .ratchet/ here, so there's nothing to do" | Harvest is the intake, not the system. `NET.md` gaps and human requests still feed the queue. |
