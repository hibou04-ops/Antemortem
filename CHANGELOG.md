# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `docs/methodology.md § Antemortem vs pre-mortem` — explicit differentiation from Gary Klein's pre-mortem (*HBR*, 2007). Tactical (code-level, solo, LLM-assisted) vs strategic (project-level, team).
- `README § How it differs from adjacent practices` — comparison table positioning antemortem against code review, pre-mortem, and postmortem on timing / scope / discharge / output axes.
- `README § Limits` — surfaces what antemortem catches vs what it does not (runtime issues, platform quirks, product wrong turns, empirical claims, blind spots) so drive-by readers see the bounds before adopting the method.
- `README § FAQ` — six common questions, including the pre-mortem confusion, model agnosticism, the two constraints that distinguish antemortem from ad-hoc "LLM, review my plan", recon budget, citation-based classification checks, and private-code usage.
- `README § Citing this work` — citation formats for the methodology and the first case study.
- `README § Contents` — updated to reflect the new sections.

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
