---
name: defining-the-risks
description: Use when no .ratchet-testing/RISKS.md exists — a missing risk model blocks ALL sizing, making this the only legal first task in a fresh deployment. Also use when the user asks to change risk tolerance ("treat exports as critical", "stop caring about the admin panel"), or when a `sizing-the-tests` escalation reveals a behavior the criteria don't cover. THE one human-gated skill in the system: risk tolerance is the user's judgment, exercised once, on record.
---

# Defining the Risks

Build and amend `RISKS.md` — the human-approved risk model that makes every later classification mechanical. Judgment happens here, once, on record; `sizing-the-tests` afterwards only looks things up.

**Prevents:** failure mode #3 (criticality inversion).

## When this fires

| Signal | Action |
|---|---|
| No `.ratchet-testing/RISKS.md` | Draft the full model (Steps 1–4). Blocks all sizing — `using-ratchet-testing` routes here as the only legal first task. |
| User asks to change risk tolerance | Amendment (Step 5). |
| `sizing-the-tests` escalation: criteria don't cover a behavior | Amendment (Step 5) — never an improvised class. |

## Step 1 — Reconnaissance

Read the project before proposing anything. You are collecting candidates, not classifying — classes attach to behaviors in `NET.md` later, via `sizing-the-tests`.

- **User-facing behaviors** — README, routes/endpoints, CLI commands, main screens: what does a user come here to do? Which workflows are the core value, which are peripheral?
- **Money / data / auth surfaces** — payment paths, migrations and deletion code, session and auth handling, secrets. These are R3 candidates by default; finding none is a finding worth stating.
- **Failure visibility** — what breaks loudly (a 500 on login) vs. silently (a malformed export, a dropped row)? Silent failures argue for a higher class.

## Step 2 — Propose criteria and examples

Draft one row per class using the spec's tier table as the frame: R0 (Accepted), R1 (Minor), R2 (Major), R3 (Critical). For each class, write the *criteria* (what failure means to the user) and 2–4 *project-specific examples* pulled from Step 1 — generic examples classify nothing. If an example's class is genuinely arguable, put it under the higher class and flag it for the human; the approval conversation is where that argument belongs.

## Step 3 — The human gate

Present the draft to the human — inline summary plus file path — and ask for approval or edits. **Approval is THE gate**: the parallel of the parent's brief approval, amortized across every future task. R3 tasks run gate-free later precisely because the R3 *criteria* were approved here.

- Record **approver and date** in the file header on approval.
- **Nothing above R0 classification is legal against an unapproved model.** A draft `RISKS.md` sizes nothing.

## Step 4 — Write RISKS.md

Save to `.ratchet-testing/RISKS.md`:

```markdown
# Risk Model — <project>

Approved by: <name>   Approved on: <YYYY-MM-DD>
<!-- Both fields empty ⇒ DRAFT: nothing above R0 may be classified against it. -->

## Class criteria

| Class | Failure means | This project's examples |
|---|---|---|
| R0 — Accepted | no user-visible consequence | <e.g. internal build scripts; log formatting> |
| R1 — Minor | degraded but peripheral experience | <e.g. changelog page fails to render> |
| R2 — Major | a primary workflow breaks, workaround exists | <e.g. search returns stale results> |
| R3 — Critical | user locked out of core value, data loss/corruption, security or money | <e.g. login denies valid users; migration drops rows> |

## Amendment log

| Date | Change | Reason | Re-approved by |
|---|---|---|---|
```

On approval: worklog `decision` entry (format per `keeping-state`) naming the approver, then return to `using-ratchet-testing` — sizing is now unblocked.

## Step 5 — Amendment procedure

Amend the criteria or examples **in place**, then append a row to the amendment log — every row carries a logged reason.

- **R3-criteria changes require re-approval**: fill the `Re-approved by` cell with the approver's name and update the header date. Until re-approved, the *old* R3 criteria remain in force.
- R0–R2 criteria and example changes: logged reason in the amendment log suffices; leave `Re-approved by` as `—`.
- One worklog `decision` entry per amendment. If the amendment came from a `sizing-the-tests` escalation, re-run that sizing against the amended model before anything else.
- Loosening criteria does not silently un-protect behaviors: existing `NET.md` rows keep their class until `sizing-the-tests` revises them with a logged reason.

## Stop conditions

- No approved model exists and the human is unavailable → worklog `blocked`. Do not classify, do not improvise a provisional class, do not proceed "pending approval."
- Reconnaissance cannot determine what the project's core value is → ask the human before drafting. The risk model is their judgment; a guessed model approved on autopilot inverts criticality with a signature on it.
- The requested change would make a money/data/auth surface R0 → require the human to state the reason themselves; record it verbatim in the amendment log.

## Rationalization check

| Thought | Reality |
|---|---|
| "The classes are obvious — skip the approval" | Risk tolerance is the user's judgment, not yours. Ten minutes of approval buys gate-free flow for every future task. |
| "I'll classify this one behavior now, get the model approved later" | Backwards. Classification against an unapproved model is exactly the improvisation the model exists to kill. |
| "This behavior fits no class — I'll pick the closest" | A criteria gap is an amendment with a logged reason, never a private judgment call. |
| "Amending for one edge case is overkill" | The amendment is three lines in a log table. The alternative is a class that exists nowhere on record. |
