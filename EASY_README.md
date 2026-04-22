# Antemortem — Easy Start

> The short version, for people who found the main README intimidating.
> Full doc: [README.md](README.md) · 한국어 Easy: [EASY_README_KR.md](EASY_README_KR.md)

## What is this?

**A postmortem explains the wreckage. An antemortem prevents it.**

This repo is a **methodology**, not a package. Seven steps + two markdown templates that make you stress-test a planned code change *on paper, with an LLM*, before you write a single line. Catches the category of bugs that neither code review nor tests can catch — because they're baked in *before* the PR exists.

## The problem it solves

Every non-trivial change starts the same way:
1. You write a spec.
2. You jot down some risks.
3. You start coding.
4. Half a day in, you discover: **two of your "risks" were imaginary**, and **one risk you never imagined is load-bearing**.

Antemortem is step 2.5 — the missing step where you front-load the discovery *before* the expensive part.

## The seven steps, in one screen

1. **Write the spec** (one paragraph).
2. **List your own traps** (generous guesses; trap / worry / unknown).
3. **Hand it to an LLM** along with the files your change touches.
4. **The LLM classifies each trap** as `REAL` (code confirms) / `GHOST` (code disproves) / `NEW` (risk the model surfaced), each with a `file:line` citation.
5. **Revise your estimate** of P(success).
6. **Update the spec** now, before you start coding.
7. **Save the doc** — future-you will reread it when something surprises you.

Step 4 is the whole discipline. No citation = no evidence. That's the rule.

> The methodology formally defines three labels (`REAL` / `GHOST` / `NEW`). The companion [antemortem-cli](https://github.com/hibou04-ops/antemortem-cli) adds a fourth, `UNRESOLVED`, for *"no evidence either way in the provided files"* — a useful honest-ignorance label when you run the protocol via the tool rather than by hand.

## Two guardrails that make it honest

- **You enumerate traps *before* the LLM sees any code.** Otherwise the model frames your risks for you (anchoring) and just agrees with everything.
- **Every classification carries a `file:line` citation.** Otherwise the LLM can hallucinate support for whatever you want to hear.

Without both: you've just traded one kind of hand-waving for another. With both: you have a mechanical, 15-minute screening step.

## How to use this repo

No install. It's docs.

```
templates/antemortem-template.md            ← copy this, fill it in
templates/antemortem-template-enhanced.md   ← for high-stakes changes (prod, data migration, security)
examples/omega-lock-audit.md                ← the first real case study
docs/methodology.md                         ← the full protocol
```

Want the tooling that scaffolds + validates the doc automatically? That's **[antemortem-cli](https://github.com/hibou04-ops/antemortem-cli)** (`pip install antemortem`). This repo is the discipline; the CLI is the pit crew.

## Did it actually work?

Yes — see [`examples/omega-lock-audit.md`](examples/omega-lock-audit.md). On a real 15-minute antemortem:

- 1 ghost trap caught (saved ~0.5 engineer-day)
- 3 risks downgraded from 30–40% to 10–15%
- 1 new spec requirement surfaced
- Post-recon P(on-time ship) went from 55–65% → 70–78%
- Implementation took one day, 20 new tests passed on first run

## When NOT to use it

- Trivial changes (typo fix, one-line config).
- Changes whose code doesn't exist yet to read (greenfield — nothing to cite).
- You're out of time. Antemortem is cheap but not free. 15 min is 15 min.

## Limits

Antemortem is a **reading discipline**. It will NOT catch:
- Runtime-only issues (race conditions, GC, network flakiness).
- Platform quirks (cp949 terminal encoding, filesystem case sensitivity).
- Product-level wrong turns ("why are we building this at all?").
- Empirical claims the code doesn't prove or disprove.

Use pre-mortem for strategic risks. Use tests for runtime behavior. Use antemortem for *"does the code I already have actually agree with my plan?"*

## Go deeper

- Full methodology: [docs/methodology.md](docs/methodology.md)
- The 3-layer stack (methodology → CLI → first case): see [README.md § The 3-layer stack](README.md#the-3-layer-stack)
- Case study: [examples/omega-lock-audit.md](examples/omega-lock-audit.md)

License: Apache 2.0. Copyright (c) 2026 hibou.
