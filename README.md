# Antemortem

![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)
![Version](https://img.shields.io/badge/version-0.1-blue)
![Status](https://img.shields.io/badge/status-docs--only-lightgrey)

*AI-assisted pre-implementation reconnaissance for software changes.*

A postmortem is what you write after something breaks. An **antemortem** is what you do before you build. You put the planned change under stress *on paper*, use an LLM to read the existing code thoroughly, enumerate traps, classify each as real or ghost with primary-source evidence, and then revise your risk and your spec before writing a single line.

It is not a replacement for testing. It is a replacement for the first half-day of implementation that usually gets burned discovering that one of your "risks" was imaginary and one you never imagined was load-bearing.

## Contents

- [Why now](#why-now)
- [The seven steps](#the-seven-steps)
- [Does it work?](#does-it-work)
- [How to use this repo](#how-to-use-this-repo)
- [Status](#status)
- [Contributing](#contributing)
- [License](#license)
- [Colophon](#colophon)

## Why now

LLM capability has made two things cheap that used to be expensive:

1. **Reading a codebase.** A capable model can scan a dozen files, trace call patterns, and correctly identify who calls what in single-digit minutes.
2. **Enumerating failure modes.** The model can list plausible traps faster and more thoroughly than a human alone.

What stayed expensive: **writing correct code**. So if AI can cheaply front-load comprehension and risk enumeration, the rational move is to do that before the expensive phase, not during it.

## The seven steps

1. **State the change.** One paragraph. What are you planning to add, remove, or refactor?
2. **Enumerate traps.** List every hypothesized risk. Be generous. Label each "trap" / "worry" / "unknown."
3. **Hand the plan + the repo to the LLM.** Ask it to read the files your change will touch.
4. **Classify findings with primary-source citations.** For each trap: REAL (code confirms) / GHOST (code contradicts) / NEW (surfaced by the model). Cite file + line for each.
5. **Revise probabilities.** Update P(success) with new trap counts.
6. **Update the spec.** Change the spec NOW, before it feels load-bearing.
7. **Save the doc.** The value is not just the recon — it's that future-you can reread it when implementation surprises you and see which assumption broke.

Do not skip step 4. Citations are the discipline. Without them you've just traded one form of hand-waving for another.

The full methodology, including rubric and failure modes, is in [docs/methodology.md](docs/methodology.md). A copy-ready template lives in [templates/antemortem-template.md](templates/antemortem-template.md).

## Does it work?

The first case study lives in [examples/omega-lock-audit.md](examples/omega-lock-audit.md). It covers a ~15-minute antemortem run on a Python calibration framework before adding a new audit submodule.

Short version of what that antemortem caught:

- **One ghost trap.** I was sure `WalkForward` folded internally. The code showed it does not — each call is one clean evaluation. Primary-source evidence, one of seven planned risks removed. ≈0.5 engineer-day saved.
- **Three downgraded risks.** JSON serialization, memory blow-up, and iterative-round bookkeeping all had existing mitigations in the codebase. Each dropped from 30–40% to 10–15%.
- **One new requirement.** The model noticed the target-shaped object existed in three roles simultaneously. Added a `target_role` field to the spec before implementation.

Post-recon P(full spec ships on time) went from 55–65% to 70–78%. The implementation took one engineer-day; 20 new tests passed on first run.

## How to use this repo

This v0.1 ships as **docs and templates only**. Clone, read, copy the template into your own project, run the protocol with the LLM of your choice.

A CLI helper (`antemortem init <project>`) is on the roadmap but not required — the value is the discipline, not the tool.

## Status

- **v0.1** — methodology + template + one case study (omega-lock). This repository.
- **v0.2 (planned)** — `antemortem` PyPI package with CLI scaffolding, extra case studies.
- **v0.3 (speculative)** — structured schemas for antemortem docs so teams can diff them across PRs.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Short version: case studies go under `examples/` via PR, methodology revisions go against `docs/methodology.md`, primary-source citations are the bar.

Release notes in [CHANGELOG.md](CHANGELOG.md). Community expectations in [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## License

MIT. See [LICENSE](LICENSE).

## Colophon

Antemortem as a named practice was crystallized during `omega_lock.audit` development on 2026-04-18. The first case study is the recon that caught a ghost trap on the [`omega-lock`](https://github.com/hibou04-ops/omega-lock) codebase before the `omega_lock.audit` submodule was built. The recon itself was performed by Claude Opus 4.7; this repo is its synthesis.
