---
name: requesting-the-seam
description: Use the moment a behavior cannot be tested without a production-source change — no injection point, a hardwired dependency (network/DB/clock/filesystem in a constructor), global state, "I'd have to edit the source to reach this" — typically hit from `backfilling-the-gap`, `pinning-the-bug`, or `characterizing-the-behavior`. Also use when harvest finds an open seam request whose seam has since landed. Production source is read-only, always; this is the no-apologies path around that wall.
---

# Requesting the Seam

"No seam" ends the test attempt, never the conversation. When a behavior can't be reached without editing production source, write a durable seam request the main ratchet or the human can pick up, record the gap honestly in `NET.md`, and take the next queued task. Never a smuggled edit, never an apology.

**Prevents:** failure mode #10 (untestable code gets apologies).

## The refusal, restated

Production source and `.ratchet/` are **read-only, always**. You do not add the constructor arg yourself, you do not sneak a "no behavior change" refactor commit, you do not fake a seam by monkeypatching internals from the test side.

## Step 1 — Name the blocker precisely

One sentence: "`<behavior-id>` cannot be tested because `<dependency>` is `<hardwired how>`." If you can't write that sentence, you haven't hit the wall yet — try again to reach the behavior from the project test paths (a process boundary, a CLI, an API surface) before requesting anything.

## Step 2 — Name the smallest seam that would unblock it

Use the parent's `finding-seams` vocabulary, cheapest first: parameter seam, extract-the-logic seam, constructor seam, extract-and-override seam, sprout seam, wrap seam (adapter). Name exactly one — the smallest that lets a test in, signature-preserving and default-preserving. The request is credible in proportion to how little it asks for.

## Step 3 — Write the request

To `.ratchet-testing/issues/<date>-seam-<slug>.md` — this skill owns the format:

```markdown
# Seam request: <behavior-id>

Status: open · opened <date> · class <R1|R2|R3>

## Blocked behavior    <behavior ID + risk class; what cannot be tested today, one paragraph, verified against current source>
## Smallest seam       <named seam type per the parent's finding-seams + the one signature/point it touches; default-preserving>
## NOT asking for      <what this request does not license: no restructuring, no behavior change, no cleanup-in-passing>
## Unblocks            <the queued task(s) waiting — backfill / pin / characterization, by task-id if queued>
```

The `Status` line is the interface: the main ratchet or the human flips it to `landed (<date>)` or `declined (<date>)`. They never have to read anything else of ours — the request must stand alone.

Non-seam records (candidate issues from characterization or flake observation) use `<date>-issue-<slug>.md` with the same Status line convention.

## Step 4 — Record the gap

- `NET.md` via `mapping-the-net`: the behavior's row is `gap` with a pointer to the request file. A blocked pin leaves its regression window open **and recorded** — visible at every audit, not forgotten.
- Worklog `blocked` line pointing at the request (`keeping-test-state`); if this blocked your active task, snapshot it and take the next queued task.

## Step 5 — Reap at harvest

`harvesting-signals` re-checks open seam requests on every pass. A request whose seam has landed — status flipped, or the seam visible in current source — queues the unblocked task: the `backfilling-the-gap`, `pinning-the-bug`, or `characterizing-the-behavior` work that was waiting, at its original priority (a waiting pin still outranks everything).

## Stop conditions

- The behavior is reachable from project test paths after all — a coarser seam exists at a boundary you control → test there; note the finer seam as a nice-to-have in the request, or skip the request entirely.
- What's needed is a rewrite, not a seam → the request records that honestly ("no small seam exists; restructuring required"), and the class decides how loud: for R3, raise it with the human directly rather than letting it wait in the queue.
- The behavior is R0 → no request; deliberate non-protection with a recorded reason is already the honest outcome.

## Rationalization check

| Thought | Reality |
|---|---|
| "It's a two-line change to prod — faster than a request" | The no-prod-writes invariant is what makes this system safe to run liberally. Two lines spends it. |
| "I'll monkeypatch the module loader in the test — no source edit" | A patched loader is a smuggled seam with worse coupling; `hardening-the-evidence`'s checklist would fail it on sight. Request the real seam. |
| "No seam, so this behavior just can't be protected — moving on" | Moving on without a record is failure mode #10 verbatim. Ten minutes of writing makes it someone's actionable task. |
| "Nobody will ever pick the request up" | Harvest re-checks it every pass and `NET.md` shows the gap at every audit. Unwritten, it is invisible forever. |
