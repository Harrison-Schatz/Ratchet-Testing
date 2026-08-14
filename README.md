# RATCHET-TESTING

**The pawl must be load-tested.**

A ratchet holds because of its pawl. In the parent methodology, the test suite *is* the pawl — the mechanism that keeps every verified click from slipping back. Ratchet-Testing is the sub-ratchet that builds and maintains that pawl: an evidence-gated methodology for coding agents, tailored to develop software tests that form a **durable safety net** — one that proves the system works today and catches it when future changes break it — effectively, accurately, and with minimal resource.

It is built on two observations of its own:

1. **In the main ratchet, tests are evidence, not an asset.** They are written in service of a task, scoped to the change, and judged by whether the gate passes. Nobody owns the suite as a product, so the suite rots in predictable ways.
2. **A passing test proves nothing.** A test that cannot fail also passes. The parent's evidence standard — freshly-run command output — is too weak here. The only proof of a test is a *witnessed failure*.

Ratchet-Testing runs **in parallel to** the main ratchet — not as a concurrent agent session, but as a concurrent *sequence of tasks* with its own durable state, its own spine, and its own gates. It outlives any single session and is resumable at any point, by the same rules as the parent: disk beats conversation.

---

## Division of labor with the main ratchet

**Main Ratchet delivers changes. Ratchet-Testing delivers and maintains the net.**

| | Main Ratchet | Ratchet-Testing |
| --- | --- | --- |
| Tests are | evidence — "does today's change work" | an asset — "will we know when this breaks" |
| Scope | the task at hand | the suite as a whole |
| Tense | present | future |
| Writes tests to | pass its done-gate (`testing-by-default`) | harden, pin, backfill, audit, and prune the net |

The main ratchet writes the first draft of the net as a side effect of doing its job. Ratchet-Testing is the net's maintainer. Neither duplicates the other: evidence-tests belong to dev tasks; durability belongs here.

## The boundary model

Three write-zones, enforced by rule:

| Zone | Access |
| --- | --- |
| Production source, `.ratchet/` | **read-only, always** |
| Project test paths | read-write |
| `.ratchet-testing/` | read-write (its own state) |

Two consequences, both load-bearing:

- **Structural invariant: Ratchet-Testing cannot break the shipped system.** Worst case, it breaks a test. This is why its human gates can sit at the risk model rather than at every task.
- **Deconfliction by reading, not locking.** Before touching any test file, check `.ratchet/STATE.md`'s roster: test files belonging to an *active* main-ratchet task are off-limits until that task lands. Read access to `.ratchet/` is the entire coordination protocol — no locks, no messaging layer, no awareness required from the main ratchet.

When a behavior is untestable without a production-source change (no seam exists), the answer is a **seam request** — a durable issue record in `.ratchet-testing/issues/` that the main ratchet (or the human) can pick up — never a smuggled edit.

## The failure modes, and the mechanism that answers each

A mechanism is a concrete, checkable behavior — never a value statement.

| # | Failure mode | Mechanism | Skill |
| --- | --- | --- | --- |
| 1 | **Tests that can't fail** — tautologies, over-mocked tests testing the mock, assertion-free tests | No test enters the net without a recorded failure demonstration: the guarded behavior is broken (revert, stash, or mutate) and the test is watched going red; the red output lands in `evidence/` | `proving-by-failure` |
| 2 | **Coverage theater** — lines covered, behaviors unprotected | The net is measured in *behaviors*, not lines: `NET.md` maps behavior → risk class → test pointers → proof pointer → status. Coverage % never appears as a gate anywhere in this methodology | `mapping-the-net` |
| 3 | **Criticality inversion** — trivial code heavily tested, risk surfaces bare | A human-approved risk model (`RISKS.md`) precedes all sizing; battery depth is a function of risk class, by rule, not by whichever code happened to be worked on recently | `defining-the-risks`, `sizing-the-tests` |
| 4 | **Suite rot / alarm fatigue** — flakes accumulate until red means nothing | A flake is corrupted evidence: immediate quarantine strips its verdict of authority, `NET.md` marks the behavior *protection suspended*, and every quarantine must reach a disposition within budget | `quarantining-the-flake` |
| 5 | **Implementation-coupled tests** — false alarms on refactor erode trust in the net | Promotion from evidence-test to durable test runs a coupling checklist: assert observable behavior at seams, not internals; mocks only at architectural boundaries | `hardening-the-evidence` |
| 6 | **Regression without a pin** — bug fixed, nothing stops it from returning | Harvest reads the main ratchet's debugging worklogs; every root-caused fix generates a pin task; the pin is proven by temporarily reverting the fix (or mutating the fixed line) and watching the new test catch it | `harvesting-signals`, `pinning-the-bug` |
| 7 | **Suite cost creep** — slow suite → run less often → net not deployed | Every test pays rent: it must map to a behavior in `NET.md` and contribute unique protection. Duplicates, tautologies, and framework-tests are deleted; suite runtime carries a budget checked at every audit | `charging-the-rent`, `auditing-the-suite` |
| 8 | **Test intent illegibility** — a future session can't tell what a test protects | Every durable test carries a behavior ID (naming/comment convention) resolving to a `NET.md` row. A test with no behavior ID is rent-defaulting by definition | `mapping-the-net`, `charging-the-rent` |
| 9 | **Unknown blind spots** — nobody can say what the net does *not* cover | `NET.md` records deliberate non-protection (`R0`, with reason) separately from gaps; any gap at R2+ auto-queues a backfill task at harvest | `mapping-the-net`, `backfilling-the-gap` |
| 10 | **Untestable code gets apologies** — "no seam" ends the conversation | Seam requests are a defined workflow with a durable artifact, not an exception begged from the user | `requesting-the-seam` |
| 11 | **The two ratchets colliding** — both editing the same test file | Deconfliction rule (above): active main-ratchet tasks own their test files; ownership is checked by reading `.ratchet/STATE.md` before any write | `harvesting-signals` |
| 12 | **Stale trust in the map** — `NET.md` claims protection that no longer exists | The map is a claim until checked: audits sample `NET.md` rows and re-prove them; resume verifies the roster against the actual suite before continuing. Agents lie accidentally — including this one's past self | `auditing-the-suite`, `resuming-work` |

## The spine

Every testing task walks the same five beats as the parent — with two beats re-tooled:

```
ORIENT (+ HARVEST) → SIZE (against RISKS.md) → BUILD (battery-scaled) → PROVE → RECORD
```

- **ORIENT + HARVEST** — read `.ratchet-testing/STATE.md`, `NET.md`, and `RISKS.md`. If the roster lists an active task, that's a resume (`resuming-work`), not a new task. Then harvest: diff `.ratchet/` against the watermark in `HARVEST.md` and convert new signals into queued tasks by rule (see *Where work comes from*). Harvest costs one read pass; it is what makes the sub-ratchet self-feeding and session-independent.
- **SIZE** — classify the behavior(s) at stake into a risk class using `RISKS.md` (`sizing-the-tests`). Classification is mechanical: the judgment already happened, once, on record, when the risk model was approved. A task touching multiple behaviors inherits the highest class among them. One line in the worklog.
- **BUILD** — the risk class decides the battery (see tier table). The task type routes to its skill: `hardening-the-evidence`, `pinning-the-bug`, `backfilling-the-gap`, or `characterizing-the-behavior`.
- **PROVE** — the gate (`proving-by-failure`). Done means: every new or promoted test has a recorded red demonstration in `evidence/`, the full battery for the behavior's class is present, and the suite is green *after* restoration. A pass is not proof; only the witnessed failure is. No red on record, no "done" — structurally blocked, not discouraged.
- **RECORD** — update the `NET.md` row(s) (status, test pointers, proof pointer), worklog entry, roster row cleared, commit. The map is updated at the moment protection changes, not at the end of a session.

**The escape hatch is inherited:** if sizing was wrong (a "minor" behavior turns out to gate a critical workflow; a pin reveals an unprotected risk surface), stop, log it, re-run sizing at the new class, keep all work that survives. De-escalation is equally legal with a stated reason — including down to R0, recorded.

## The tiers — risk classes

The parent tiers by **task size**. Ratchet-Testing tiers by **consequence to the user**. Risk class attaches to *behaviors*, not tasks; it is assigned once in `NET.md` and revised only through `sizing-the-tests` with a logged reason.

| | **R0 — Accepted** | **R1 — Minor** | **R2 — Major** | **R3 — Critical** |
| --- | --- | --- | --- | --- |
| **Failure means** | no user-visible consequence | degraded but peripheral experience | a primary workflow breaks, workaround exists | user locked out of core value, data loss/corruption, security or money |
| **Looks like** | dev-only tooling, log formatting, internal scripts | changelog display fails to render | search returns stale results; export produces a malformed file | login denies valid users; payment double-charges; migration drops rows |
| **Battery** | none — a *recorded decision* with reason, not an omission | happy-path smoke + one failure path | happy + failure paths + key edge cases | characterization pin + happy + failure + edges + invariant/property tests where feasible |
| **Proof standard** | n/a | red demo per test | red demo per test | red demo per test **+** membership in the mutation-audit sample every sweep |
| **Audit frequency** | n/a | rarely sampled | sampled | sampled **every** audit sweep |
| **Human gates** | none (visible in `NET.md`) | none | none | none per task — the R3 *criteria themselves* were human-approved in `RISKS.md` |

Two rules keep the classes honest:

- **R0 is legal and loud.** Deliberate non-protection recorded with a reason is minimality; accidental non-protection is a gap. `NET.md` never confuses the two.
- **"Less testing" is never "no testing" above R0.** The changelog-display failure is not critical — but it still gets its smoke test. (R1 exists precisely so peripheral behaviors get *proportionate* protection instead of zero.)

## Where work comes from — the harvest

Ratchet-Testing has no inbox. Because it may only *read* `.ratchet/`, all work is **pulled, never pushed** — derived by reading the main ratchet's artifacts against a watermark (`HARVEST.md` records the last-read position). The main ratchet requires zero awareness that this system exists; the methodology degrades gracefully if either side is absent.

| Signal (read from) | Generates | Task type |
| --- | --- | --- |
| Tier 2/3 task lands (`.ratchet/STATE.md` roster, worklogs) | reinforcement pass over the landed change's evidence-tests | `hardening-the-evidence` |
| Root-caused fix (`.ratchet/worklog/`, debugging entries) | regression pin | `pinning-the-bug` |
| New rule in `.ratchet/LESSONS.md` | candidate invariant to encode as a test | `backfilling-the-gap` |
| Declined/deferred review finding (`.ratchet/review/`, `.ratchet/` issues) | risk-register entry; backfill if it names a behavior | `backfilling-the-gap` |
| Brief acceptance checks (`.ratchet/briefs/`) | candidate durable tests for behaviors entering the map | `mapping-the-net` → `backfilling-the-gap` |
| Gap at R2+ discovered in `NET.md` | backfill | `backfilling-the-gap` |
| Scheduled sweep due | suite health audit | `auditing-the-suite` |
| Human request | as specified | any |

Queue ordering is rule-governed: **pins first** (a regression window is open until pinned), then R3 gaps, then hardening, then lower-class backfill, then audits — preempted only by an explicit human request.

## The flake protocol

A flake is not a broken test. It is **corrupted evidence** — worse than no evidence, because it degrades the authority of every green run and trains humans and agents to ignore red. The protocol:

1. **Quarantine on first confirmed flake.** An entry in `FLAKES.md` records the test, the behavior ID, observed verdict pattern, and reproduction context. The test's verdict is immediately stripped of authority (excluded from the gate-relevant set) but keeps executing for data.
2. **The map never overstates the net.** The behavior's `NET.md` row flips to *protection suspended* the moment its test is quarantined. A suspended R2+ behavior is a gap, and gaps at R2+ queue backfill.
3. **Every quarantine reaches a disposition within budget** — three testing sessions or the next audit sweep, whichever comes first:
   - **Deflake** — root-cause the nondeterminism, fix the test, prove stability by N consecutive clean runs under repetition/stress, restore authority;
   - **Rewrite** — the test's approach was unsound; replace it (new test walks `proving-by-failure` like any other);
   - **Delete** — and record the resulting state honestly in `NET.md` (gap or R0-with-reason).
4. **An overdue disposition blocks the sweep.** The next `auditing-the-suite` run cannot be declared clean while an out-of-budget quarantine exists. Rot is made structurally loud instead of quietly cumulative.

## Principles

The parent's seven principles are inherited whole — disk beats conversation, evidence is the currency, plans are hypotheses, ceremony pays rent, tests-not-religion, the codebase you have, agents lie accidentally. Ratchet-Testing adds five of its own:

1. **A passing test proves nothing. Only a witnessed failure proves a test.** The pawl must be load-tested before weight is put on it.
2. **The net is measured in behaviors, not lines.** A coverage percentage is an input at best and a vanity metric at worst; it is never a gate.
3. **Consequence allocates effort, not activity.** What gets tested hardest is what hurts users most when it breaks — not whatever was edited last.
4. **Every test pays rent; absence is either deliberate or a gap.** Deleting a test is a first-class, recorded outcome. So is deciding not to write one.
5. **Corrupted evidence is worse than no evidence.** One tolerated flake taxes the credibility of the entire suite.

## What Ratchet-Testing refuses to do

- **It refuses to chase a coverage number.** Behaviors protected, at proof standard, is the only score.
- **It refuses to test everything equally.** R0 exists, visibly, with reasons — minimal resource is a design goal, not a compromise.
- **It refuses to trust a green run as proof of a new test.** Red first, on record, or it isn't in the net.
- **It refuses to touch production source.** Seams are requested through a durable artifact, never smuggled in.
- **It refuses to let a flake keep voting.** Quarantine is immediate; disposition is budgeted; overdue rot blocks the clean bill.
- **It refuses to duplicate the main ratchet.** Evidence-tests belong to dev tasks. This system hardens, pins, backfills, audits, and prunes.
- **It refuses to trust its own map.** `NET.md` is a claim; audits and resumes check it against the repo.

## Divergences from the parent Ratchet — with reasons

| Divergence | Reason |
| --- | --- |
| **Tiered by risk-to-user, not task size** | Test effort should follow consequence, not activity. A one-line login test outweighs a fifty-line changelog battery. |
| **Evidence standard upgraded from "command output" to "witnessed failure"** | The parent's standard cannot distinguish a protective test from a tautology; both print green. |
| **Central artifact is a map (`NET.md`), not a roster** | The parent's product is completed tasks; this system's product is the net's *shape* — including its known holes. |
| **Work is harvested, not assigned** | Read-only access to `.ratchet/` forbids push. Pull-based intake with a watermark keeps the systems decoupled and both resumable. |
| **Three-zone write model with a structural no-prod-writes invariant** | The sub-system must be safe to run liberally; incapable-of-breaking-prod is what buys gate-free task flow. |
| **Deletion is a spine outcome, not an exception** | "Minimal resource" is a stated goal; a net you can't afford to run protects nothing. |
| **Human gates sit at the risk model, not per task** | Risk tolerance is the user's judgment; it is exercised once, on record, in `RISKS.md` — then classification is mechanical. Parallel to the parent's brief approval, amortized across every future task. |

## The state directory

Everything a fresh session needs lives in `.ratchet-testing/` at the repo root:

```
.ratchet-testing/
├── STATE.md      # roster of active testing tasks (one row each) — id, class, phase, NEXT ACTION, pointers
├── NET.md        # THE asset: behavior → risk class → test pointers → proof pointer → status (protected / suspended / gap / R0+reason)
├── RISKS.md      # human-approved risk model: class criteria, project-specific examples, amendment log
├── HARVEST.md    # watermark + last-read pointers into .ratchet/
├── FLAKES.md     # quarantine register: entry, verdict pattern, budget clock, disposition
├── state/        # <task-id>.md — per-task cold-resume snapshot
├── worklog/      # <task-id>.md — append-only journal: sizings, decisions, surprises, evidence pointers
├── evidence/     # red demonstrations and mutation-audit reports, by behavior ID
└── issues/       # seam requests and durable records readable by the main ratchet (which never has to read them)
```

Resume cost is at most one step, same as the parent: read `STATE.md`, verify claims against `git status` and an actual suite run (state is a claim until checked), reconcile, continue. If `NET.md` grows past comfortable single-file size, it becomes an index over `net/<area>.md` — deferred until it hurts.

## The skill catalog

Load **`using-ratchet-testing`** first — it routes every task and makes the rest predictable.

**Foundation**

| Skill | Defeats |
| --- | --- |
| `defining-the-risks` | criticality inversion — builds and amends `RISKS.md`; the one human-gated skill |
| `mapping-the-net` | coverage theater; unknown blind spots; intent illegibility — owns `NET.md` and the behavior-ID convention |

**The spine**

| Skill | Defeats |
| --- | --- |
| `harvesting-signals` | starvation and collision — intake by reading `.ratchet/`; owns the watermark, the queue rules, and the deconfliction check |
| `sizing-the-tests` | mis-sized batteries — mechanical classification against `RISKS.md`; the escalation escape hatch |
| `proving-by-failure` | tests that can't fail — the gate; owns the red-demonstration mechanics (revert / stash / mutate) and `evidence/` |
| `landing-the-tests` | protection claimed but never integrated — commit conventions, `NET.md` update atomicity |

**Task types (BUILD routes to one)**

| Skill | Defeats |
| --- | --- |
| `hardening-the-evidence` | implementation-coupled tests — promotes task-scoped evidence-tests into durable behavior tests via the coupling checklist |
| `pinning-the-bug` | regression without a pin — turns every root-caused fix into a permanent catch |
| `backfilling-the-gap` | known-unprotected R2+ behaviors — closes recorded gaps in priority order |
| `characterizing-the-behavior` | untested legacy behavior — pins current behavior at a seam before anyone changes it (test-writes only; inherits the parent's discipline within this system's boundary) |

**Maintenance**

| Skill | Defeats |
| --- | --- |
| `auditing-the-suite` | stale trust in the map; suite cost creep — mutation-samples `NET.md` claims (R3 every sweep), checks the runtime budget, censuses redundancy |
| `quarantining-the-flake` | alarm fatigue — the flake protocol, end to end |
| `charging-the-rent` | resource waste — the deletion lens; every test maps to a behavior or leaves |
| `requesting-the-seam` | untestable-code apologies — durable seam-request records in `issues/` |

**Inherited pattern, re-scoped**

| Skill | Defeats |
| --- | --- |
| `keeping-state` / `resuming-work` | context loss between sessions — the parent's discipline, pointed at `.ratchet-testing/` |

Sixteen skills. Each declares which failure mode it prevents; a skill that prevents nothing gets deleted.

## Open questions

Deliberately unresolved — each needs a decision, not more prose:

1. **Harvest cadence.** Every session start (cheap, always fresh) vs. on-demand when the queue empties. Leaning session-start: one read pass is nearly free and staleness is the greater risk.
2. **Sweep cadence.** Calendar-based, every-N-harvests, or triggered by suite-runtime drift past budget. Mutation audits are the expensive item; sampling rate per class is the real knob.
3. **Deployment interleaving.** The same agent alternating dev tasks and testing tasks, vs. dedicated testing sessions. The interface is actor-agnostic by design, so this stays a deployment choice — but the queue-priority rules may want tuning per mode.
4. **Behavior granularity.** What is one row in `NET.md` — a user-visible capability ("valid user can log in"), or finer ("login rejects expired tokens")? Proposal: capability-level rows with check-level sub-entries, mirroring brief → acceptance-check structure so harvested briefs map cleanly.
5. **Flake budget size.** Three sessions is a placeholder; the right number is "short enough that quarantine feels urgent, long enough that dispositions aren't rushed."

## Provenance

Ratchet-Testing is a tailored sub-ratchet of [Ratchet](https://github.com/Harrison-Schatz/Ratchet), inheriting its spine shape, its state discipline, and its principles — then re-tooling the two beats where testing demands more: sizing (by consequence, not size) and verification (by witnessed failure, not green output). Every divergence from the parent has a written reason. If you can't predict which skill fires for a given signal from this document and `using-ratchet-testing` alone, that's a bug — file it.
