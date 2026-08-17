# Changelog

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
