# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.2] - 2026-04-22

### Changed

- **Relicense MIT → Apache License 2.0.** Aligns Antemortem with the rest of the hibou04-ops stack (`omega-lock`, `antemortem-cli`, `omegaprompt`, `mini-omega-lock`, `mini-antemortem-cli`). Adds explicit patent grant and trademark preservation. `NOTICE` file added per Apache 2.0 § 4(d).
- `README` / `README_KR` — License badge + License section updated.

## [0.1.1] - 2026-04-21

### Added

- `docs/methodology.md § Antemortem vs pre-mortem` — explicit differentiation from Gary Klein's pre-mortem (*HBR*, 2007). Tactical (code-level, solo, LLM-assisted) vs strategic (project-level, team).
- `docs/methodology.md § Enhanced protocol (optional)` — four additions for high-stakes changes: calibration dimensions (P, evidence strength, blast radius, reversibility), fine-grained classification subtypes (REAL-structural, REAL-runtime-uncertain, GHOST-mitigated, GHOST-unreachable, GHOST-assumption-error, GHOST-test-covered, NEW-spec-gap, NEW-coupling, NEW-operational, NEW-policy), explicit skeptic pass (step 4b), and decision-first output structure (five blocks: blockers / spec mutations / safe path / runtime validation / deprioritized).
- `templates/antemortem-template-enhanced.md` — enhanced template, a superset of the basic one. Adds calibration columns to the trap table, fine-grained classification, skeptic pass section (§4b), five-block decision document (§6), and prediction-feedback table (§8) for building a personal track record of recon-vs-reality outcomes. Philosophy header + counter-hypothesis patterns included.
- `README § How it differs from adjacent practices` — comparison table positioning antemortem against code review, pre-mortem, and postmortem on timing / scope / discharge / output axes.
- `README § Limits` — surfaces what antemortem catches vs what it does not (runtime issues, platform quirks, product wrong turns, empirical claims, blind spots) so drive-by readers see the bounds before adopting the method.
- `README § FAQ` — six common questions, including the pre-mortem confusion, model agnosticism, the two constraints that distinguish antemortem from ad-hoc "LLM, review my plan", recon budget, citation-based classification checks, and private-code usage.
- `README § Citing this work` — citation formats for the methodology and the first case study.
- `README § Contents` — updated to reflect the new sections.
- `README` — enhanced-template pointer added to the protocol blurb so readers know when to reach for the richer form.

## [0.1] - 2026-04-20

Initial public release. Docs, template, and first case study.

### Added

- `docs/methodology.md` — seven-step protocol, rubric, limits, practical notes, when-NOT-to-antemortem.
- `templates/antemortem-template.md` — eight-section copy-ready skeleton for running the protocol on any change.
- `examples/omega-lock-audit.md` — first case study: a ~15-minute antemortem on the [`omega-lock`](https://github.com/hibou04-ops/omega-lock) Python calibration framework before adding its `omega_lock.audit` submodule. Ghost-trap detection (walk-forward folding), three downgraded risks (JSON serialization, memory blow-up, iterative-round bookkeeping), one new spec requirement (`target_role`), and a post-implementation honest-miss note (cp949 em-dash terminal encoding).
- `README.md` — thesis, summary of the seven steps, case-study teaser, status, contributing pointer.
- `CONTRIBUTING.md` — submission bar for case studies and methodology revisions.
- `CODE_OF_CONDUCT.md` — Contributor Covenant v2.1 reference.
- `.github/` — issue template for documentation issues, PR template with case-study checklist.
- `LICENSE` — MIT.

### Notes

- v0.1 is **docs and templates only**. No Python code, no CLI, no installable package.
- Python CLI (`antemortem init`, `antemortem run`) is on the v0.2 roadmap. v0.3 (speculative) explores structured schemas for antemortem docs.
