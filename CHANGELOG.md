# Changelog

## 0.1.3 — 2026-09-02

### Changed
- The deconfliction check reads each active main-ratchet task's `.ratchet/state/<task-id>.md` for owned paths, which the parent's roster does not carry (`harvesting-signals`, README). (#6)
- `pinning-the-bug` intake names the parent's actual trail — the `hypothesis:` line and `evidence` entry in the worklog, with the fix commit in the task's `done` entry. (#6)
- The harvest reads `.ratchet/plans/` and `.ratchet/issues/` as sources, and the watermark position may be a timestamp when `.ratchet/` is not version-controlled (`harvesting-signals`, README). (#6)
- The roster row is an index with `state file` and `worklog` columns; NEXT ACTION lives only in the state file (`keeping-test-state`, README). (#6)
- `.ratchet-testing/issues/` is defined as what testing finds, distinct from the parent's `.ratchet/issues/` for problems found developing the application, which this system reads and never writes (`requesting-the-seam`, `keeping-test-state`, README). (#6)
- Declined review findings and open `.ratchet/issues/` records both route to `mapping-the-net` then `backfilling-the-gap` (`using-ratchet-testing`, `harvesting-signals`). (#6)

## 0.1.2 — 2026-09-02

### Changed
- Removed two instructions a prompt audit found dated for current models. `resuming-test-work` no longer caps the resume summary at "2–4 sentences" — current models already under-narrate, so a sentence count cut the wrong way; the step now asks for a summary written for a reader who did not see the previous session, same content list. `characterizing-the-behavior` dropped the "assert a wrong-but-plausible value first" aside — method coaching that displaced the model's own approach without changing what the step requires. Audit scope was the 17 skills; everything else that matched a dated idiom was judged load-bearing and kept.

## 0.1.1 — 2026-08-17

### Changed
- Renamed the two inherited skills that collided with the parent Ratchet catalog: `keeping-state` → `keeping-test-state` and `resuming-work` → `resuming-test-work`. Installing both catalogs into one skills directory previously let whichever repo copied second silently shadow the other's version, breaking one system's context-loss guarantee. The rename follows the convention every other inherited skill already used (`sizing-the-tests`, `landing-the-tests`, …); references to "the parent's `keeping-state`/`resuming-work`" intentionally keep the parent names.
- Tightened `resuming-test-work`'s trigger description: a bare "continue" now routes to the parent's `resuming-work`; only testing-flavored resumes route here.

## 0.1.0 — 2026-08-14

First release: the Ratchet-Testing methodology, v0.

### Added
- The Ratchet-Testing v0 specification (`README.md`) defined evidence-gated test-suite maintenance for coding agents: only a witnessed failure proves a test, and effort follows risk-to-user, not activity.
- A 17-skill catalog under `skills/` covered the full lifecycle — routing, foundation, the five-beat spine, four task types, maintenance, and inherited state/resume discipline — entered via `using-ratchet-testing`.
- The spec established the operating contract adopters set up: a three-zone write model that cannot touch production source, pull-based harvest intake from `.ratchet/`, a flake quarantine protocol, and the `.ratchet-testing/` state directory.
- Skill routing shipped verified — a three-lens adversarial review applied 44 findings before landing, ending at a 15/15 routing-predictability audit.
