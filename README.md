# Antemortem

[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.1-blue)](CHANGELOG.md)
[![Status](https://img.shields.io/badge/status-stable-brightgreen)](#status)
[![Tooling](https://img.shields.io/badge/tooling-antemortem--cli-blueviolet)](https://github.com/hibou04-ops/antemortem-cli)

> **AI-assisted pre-implementation reconnaissance for software changes.**
> A postmortem is what you write after something breaks. An antemortem is what you do before you build — and the discipline that keeps it honest.

A postmortem explains the wreckage. An antemortem prevents the half-day of implementation that usually gets burned discovering that one of your "risks" was imaginary and one you never imagined was load-bearing. You put the planned change under stress *on paper*, hand the spec and the implicated files to a capable LLM, classify every hypothesized risk as `REAL` / `GHOST` / `NEW` / `UNRESOLVED` with primary-source `file:line` citations, and revise the spec *before* you write a single line.

This repo holds the methodology — the seven-step protocol, two templates, and the first case study. The companion tool [`antemortem-cli`](https://github.com/hibou04-ops/antemortem-cli) automates the scaffolding, the classification pass, and the schema lint (Pydantic-enforced, citations re-verified on disk). Together they form a three-layer stack: methodology here, tooling there, first shipped artifact at [`omega-lock`](https://github.com/hibou04-ops/omega-lock).

## Contents

- [Why this discipline exists](#why-this-discipline-exists)
- [The seven steps](#the-seven-steps)
- [Does it work?](#does-it-work)
- [How it differs from adjacent practices](#how-it-differs-from-adjacent-practices)
- [The 3-layer stack](#the-3-layer-stack)
- [Limits](#limits)
- [How to use this repo](#how-to-use-this-repo)
- [Status](#status)
- [FAQ](#faq)
- [Contributing](#contributing)
- [Citing this work](#citing-this-work)
- [License](#license)
- [Colophon](#colophon)

## Why this discipline exists

LLM capability has made two things cheap that used to be expensive:

1. **Reading a codebase.** A capable reasoning model can scan a dozen files, trace call patterns, and correctly identify who calls what in single-digit minutes.
2. **Enumerating failure modes.** The model can list plausible traps faster and more thoroughly than a human alone.

What stayed expensive: **writing correct code**. If AI can cheaply front-load comprehension and risk enumeration, the rational move is to do that *before* the expensive phase, not during it. Antemortem is the discipline that turns that capability into a repeatable protocol — with guardrails against the failure modes of asking an LLM to review a plan (anchoring, vibes-based agreement, hallucinated evidence).

Two guardrails keep the discipline honest:

1. **You enumerate your own traps before the model sees any code.** Prevents anchoring on the model's framing.
2. **Every classification carries a `file:line` citation.** "The code shows X" is not evidence; *"line 82 of `walk_forward.py` calls `evaluate()` once per params, with no surrounding loop"* is. Without this, you have traded one form of hand-waving for another.

## The seven steps

1. **State the change.** One paragraph. What are you planning to add, remove, or refactor?
2. **Enumerate traps.** List every hypothesized risk. Be generous. Label each `trap` / `worry` / `unknown`.
3. **Hand the plan + the repo to the LLM.** Ask it to read the files your change will touch.
4. **Classify findings with primary-source citations.** For each trap: `REAL` (code confirms) / `GHOST` (code contradicts) / `NEW` (surfaced by the model) / `UNRESOLVED` (no evidence either way). Cite `file:line` on every non-UNRESOLVED label.
5. **Revise probabilities.** Update `P(success)` with new trap counts.
6. **Update the spec.** Change the spec NOW, before it feels load-bearing.
7. **Save the doc.** The value is not just the recon — it's that future-you can reread it when implementation surprises you and see which assumption broke.

Do not skip step 4. Citations are the discipline.

The full methodology, including rubric and failure modes, is in [`docs/methodology.md`](docs/methodology.md). A copy-ready template lives in [`templates/antemortem-template.md`](templates/antemortem-template.md).

For high-stakes changes (prod deploys, data migrations, security boundaries), use the [enhanced template](templates/antemortem-template-enhanced.md) — it adds calibration dimensions (evidence strength, blast radius, reversibility), fine-grained classification subtypes, an explicit skeptic pass, and a decision-first output structure. See [`docs/methodology.md § Enhanced protocol`](docs/methodology.md#enhanced-protocol-optional) for the rationale.

## Does it work?

The first case study lives in [`examples/omega-lock-audit.md`](examples/omega-lock-audit.md). It covers a ~15-minute antemortem run on a Python calibration framework before adding a new audit submodule.

Short version of what that antemortem caught:

- **One ghost trap.** I was sure `WalkForward` folded internally. The code showed it does not — each call is one clean evaluation. Primary-source evidence, one of seven planned risks removed. ≈0.5 engineer-day saved.
- **Three downgraded risks.** JSON serialization, memory blow-up, and iterative-round bookkeeping all had existing mitigations in the codebase. Each dropped from 30–40% to 10–15%.
- **One new requirement.** The model noticed the target-shaped object existed in three roles simultaneously. Added a `target_role` field to the spec before implementation.

Post-recon `P(full spec ships on time)` went from 55–65% to 70–78%. The implementation took one engineer-day; 20 new tests passed on first run.

The same doc includes an honest post-implementation note on what the recon *missed* (a Windows cp949 em-dash terminal encoding issue that surfaced at runtime). Antemortems don't catch platform encoding issues — see [Limits](#limits).

## How it differs from adjacent practices

| Practice | Timing | Scope | Typical discharge | Output |
|---|---|---|---|---|
| Code review | After the change is written | Diff-local | Minutes, per PR | Approval / change requests |
| **Antemortem** | **Before** the change is written | Change-local | 15–30 min, solo + LLM | Risk-classified doc with `file:line` citations |
| Pre-mortem (Klein, 2007) | Before the project commits | Project-level | 30–60 min, team exercise | Ranked failure scenarios |
| Postmortem | After something broke | Incident-local | Hours, team | Root cause + action items |

Code review catches what is on the PR. An antemortem catches what is *not yet* on the PR — because the author has not written it yet. Pre-mortem is strategic (*should we do this?*); antemortem is tactical (*given that we will, what does the code already tell us about the risks of doing it this way?*). See [`docs/methodology.md § Antemortem vs pre-mortem`](docs/methodology.md#antemortem-vs-pre-mortem) for the longer distinction.

## The 3-layer stack

Antemortem does not exist in isolation. It is the middle tier of a layered discipline:

```
 Layer 3 │  antemortem-cli  (tooling)      Scaffold, classify, lint
         │  v0.2.0 on PyPI                 https://github.com/hibou04-ops/antemortem-cli
         │      operationalizes
 Layer 2 │  Antemortem  (this repo)        Methodology, templates, case studies
         │  v0.1.1                         https://github.com/hibou04-ops/Antemortem
         │      demonstrated by
 Layer 1 │  omega-lock  (first case)       Python calibration audit framework
         │  v0.1.4 on PyPI                 https://github.com/hibou04-ops/omega-lock
```

- **Layer 1 — [omega-lock](https://github.com/hibou04-ops/omega-lock)** is the Python calibration framework that Antemortem was *first practiced on*. Its `omega_lock.audit` submodule was built using a 15-minute antemortem recon that caught the ghost trap documented above.
- **Layer 2 — Antemortem** (this repo) is the methodology extracted from that build: the seven-step protocol, the basic and enhanced templates, the first case study. Docs-only by design — the discipline has to be legible without running anything.
- **Layer 3 — [antemortem-cli](https://github.com/hibou04-ops/antemortem-cli)** is the tooling that removes the friction: `antemortem init`, `antemortem run`, `antemortem lint`. Pydantic-enforced schema, disk-verified citations, CI-ready lint gate.

The layering matters for correctness: the methodology was validated by a real shipped artifact *before* the CLI was built, so the tool is automating a protocol that is already known to work — not a protocol invented alongside the tool.

## Limits

An antemortem is a **reading discipline**. What it catches:

- Ghost traps whose disproof lives in the code itself.
- Missing or wrong spec fields the recon surfaces.
- Hidden but visible-from-source mitigations you had forgotten about.
- Call patterns that differ from your mental model.

What it does **not** catch:

- Runtime-only issues (race conditions, GC timing, network behaviour).
- Platform-specific issues the model has no file-level evidence for (text encoding, shell quirks, OS-specific path handling).
- Product-level wrong turns — a spec can be internally consistent and still build the wrong thing.
- Claims requiring empirical measurement (performance, numerical stability, accuracy on real data).
- Your own blind spots, if you only ask the model about the risks you already worry about.

The world an antemortem knows is the world in the files. Anything outside that, it will speak about with misplaced confidence if you let it. Full discussion in [`docs/methodology.md § Limits`](docs/methodology.md#limits).

## How to use this repo

This repo ships as **docs and templates only**. Clone, read, copy the template into your own project, run the protocol with the LLM of your choice.

For programmatic automation — scaffolding, LLM invocation with prompt caching, Pydantic-enforced classifications, and disk-verified citation lint — install the companion CLI:

```bash
pip install antemortem
```

See [`antemortem-cli`](https://github.com/hibou04-ops/antemortem-cli) for the full tooling.

## Status

- **v0.1.1 (current)** — methodology + basic template + enhanced template + first case study (omega-lock). Stable.
- **v0.2 — shipped as [`antemortem-cli`](https://github.com/hibou04-ops/antemortem-cli)**. CLI + PyPI + Pydantic schemas + disk-verified citation lint. Released 2026-04-22.
- **v0.3 (planned)** — structured schemas for antemortem docs so teams can diff them across PRs. Experimental track.

Full changelog: [`CHANGELOG.md`](CHANGELOG.md).

## FAQ

**Isn't this just pre-mortem with a new name?**
No. Gary Klein's pre-mortem (*HBR*, 2007) is a team-level strategic exercise. Antemortem is change-level, solo, source-code-grounded, and discharged in 15–30 minutes. See [`docs/methodology.md § Antemortem vs pre-mortem`](docs/methodology.md#antemortem-vs-pre-mortem).

**Which LLM do I need?**
Any reasoning model that can follow multi-file call chains. The discipline is model-agnostic; the engine is interchangeable. The first case study and the `antemortem-cli` tooling both pin a specific Claude model for reproducibility, but you can adapt the protocol to any sufficiently capable reasoner.

**How is this different from just asking an LLM "review my plan"?**
Two constraints. Step 2 forces you to enumerate your own risks before the LLM sees them (prevents anchoring on its framing). Step 4 forces `file:line` citations (prevents accepting its vibes). Without these, you have traded one form of hand-waving for another.

**Is 15 minutes realistic for a 1–2 day change?**
For a change touching 4–7 files that the model can parallel-read, yes. For larger changes, split into multiple antemortems. If the recon exceeds an hour, the change itself is probably too big — split it first.

**What if the LLM's classification is wrong?**
The citations are the check. *"The code shows X"* is not evidence; *"line 82 of `walk_forward.py` calls `evaluate()` once per params, no loop"* is. Verify each citation against the actual file. If the citation is wrong, the classification is wrong. `antemortem-cli lint` automates this verification.

**Can I run an antemortem on closed-source or private code?**
Yes. The LLM reads what you give it; it does not need a public repo. The only constraint is that citations in a published case study should quote enough inline context to be verifiable by readers without repo access.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). Short version: case studies go under `examples/` via PR, methodology revisions go against `docs/methodology.md`, primary-source citations are the bar.

Release notes in [`CHANGELOG.md`](CHANGELOG.md). Community expectations in [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).

## Citing this work

If you reference this methodology in a paper, blog post, or internal document:

```
Antemortem v0.1.1 — AI-assisted pre-implementation reconnaissance for software changes.
https://github.com/hibou04-ops/Antemortem, 2026.
```

The first case study may be cited separately:

```
Antemortem case study: omega_lock.audit (2026-04-18).
https://github.com/hibou04-ops/Antemortem/blob/main/examples/omega-lock-audit.md
```

The companion tooling:

```
antemortem-cli v0.2.0 — tooling for the Antemortem pre-implementation reconnaissance discipline.
https://github.com/hibou04-ops/antemortem-cli, 2026.
```

## License

MIT. See [LICENSE](LICENSE).

## Colophon

Antemortem as a named practice was crystallized during `omega_lock.audit` development on 2026-04-18. The first case study is the recon that caught a ghost trap on the [`omega-lock`](https://github.com/hibou04-ops/omega-lock) codebase before the `omega_lock.audit` submodule was built. The recon itself was performed with Claude inside Claude Code; this repo is its synthesis.
