---
name: proving-by-failure
description: Use BEFORE flipping any NET.md row to `protected`, BEFORE claiming any new, promoted, or rewritten test protects anything, and BEFORE any testing task says "done" — this is the PROVE beat of every spine walk, for all four task types (harden, pin, backfill, characterize). Also use when re-proving a sampled row during an audit, or whenever anyone asks "does this test actually catch anything?". A pass is not proof; only a witnessed failure is.
---

# Proving by Failure

Green output cannot distinguish a protective test from a tautology — both print the same thing. The only proof that a test guards a behavior is a witnessed failure: the behavior broken, the test red, the break undone, the test green again, all on record. This gate is structural: no red in `evidence/`, no "done" — there is nothing to argue about.

**Prevents:** failure mode #1 (tests that can't fail).

## The red-demo procedure

Run this for EVERY new or promoted test — not a sample.

1. **Identify** the guarded behavior — its `<area>/<slug>` or `<area>/<slug>#<check-slug>` ID — and the test that claims to guard it.
2. **Break the behavior** by exactly ONE of these, and record which:
   - **revert** the fix commit: `git revert -n <sha>` (or `git stash apply` of a saved breaking patch);
   - **stash** the implementation hunk;
   - **mutate** the guarded line (e.g. flip `<` to `<=`; note file:line and the exact edit).
3. **Run the test.** Capture the verbatim red output. It must fail on the assertion that names the behavior — an import error or crash proves the test ran, not that it guards.
4. **Restore**: `git restore <paths>` / `git stash pop` / undo the mutation. Prove it: `git status` clean, `git diff` EMPTY across production source. The check goes into the entry.
5. **Re-run: green.** Same test, restored tree.
6. **Append both outputs, dated,** to `.ratchet-testing/evidence/<area>--<slug>.md`, and write a worklog `proof` line pointing at the entry — pointers, never pasted output (`keeping-test-state`).

**On the write zone:** step 2 is the one sanctioned touch of production source — working tree only, never committed, proven undone in step 4 before anything else happens. A dirty tree after step 4 is an emergency, not a detail: restore before any other action.

## Evidence entry format

This skill owns `.ratchet-testing/evidence/` — append-only files, one per behavior, the ID's `/` flattened to `--`. Entries are red demos, restore-green confirms, and mutation-audit results (`auditing-the-suite` appends that last kind). Red-demo entry:

```
## 2026-08-14 — red demo — <task-id>
test: tests/auth/login.test.ts::rejects_expired_token [net: auth/valid-login#expired-token]
break: mutate — src/auth/token.ts:88, `<` → `<=` (working tree only)
red:  AssertionError: expected 401, got 200  (exit 1)
restore: git restore src/auth/token.ts — git status clean, git diff empty
green: 1 passed (exit 0)
```

Verbatim means the assertion line(s) and exit status — enough for an audit to re-run the same break from this entry alone. An R3 entry WILL be re-run: record the mutation exactly.

## Done criteria — all four, checkable

- [ ] Red demo on record in `evidence/` for every new or promoted test in the task.
- [ ] Battery complete for the behavior's risk class, per the `sizing-the-tests` table.
- [ ] Full suite green after restoration — the whole suite, not just the new test.
- [ ] `NET.md` row update staged with a proof pointer into `evidence/` — it lands atomically with the tests via `landing-the-tests`, never before.

**R3 note:** proof at R3 does not end here. Every R3-protected test joins EVERY mutation-audit sample, so this red demo will be re-witnessed each sweep (`auditing-the-suite`).

## Stop conditions

- **The test won't go red** under any of the three breaks → it cannot fail; it does not enter the net. Rewrite it — or, if the behavior is unreachable without a production-source change, `requesting-the-seam`.
- **Red for the wrong reason** (error, not the behavior's assertion) → not proof. Fix the test, re-run the procedure.
- **Restoration not clean** (`git diff` non-empty) → stop everything and restore; nothing else happens on a dirty tree.
- **Suite not green after restoration** → something beyond the demo broke; do not proceed to `landing-the-tests`.

## Rationalization check

| Thought | Reality |
|---|---|
| "It passed on the first run — clearly it works" | A tautology also passes on the first run. |
| "I watched it fail during development" | Scrollback dies with the session. If it isn't in `evidence/`, the net never saw it. |
| "Mutating prod violates the write zone" | The break is working-tree only, never committed, proven undone. That is the sanction — and its whole extent. |
| "Six steps per test is ceremony" | The entry is ~8 lines, and it is what every audit and resume trusts. Ceremony that pays rent stays. |
