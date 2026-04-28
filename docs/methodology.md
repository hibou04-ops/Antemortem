# Antemortem: AI-assisted Pre-implementation Reconnaissance for Software Changes

| | |
|---|---|
| **Author** | Kyunghoon Gwak (곽경훈) — [@hibou04-ops](https://github.com/hibou04-ops), `hibouaile04@gmail.com` |
| **First publication** | 2026-04-20 (commit `7e378ad`) |
| **Current version** | v0.1.1 (Apache 2.0, relicensed 2026-04-22, commit `8bf43b4`) |
| **Repository** | https://github.com/hibou04-ops/Antemortem |
| **Companion tool** | https://github.com/hibou04-ops/antemortem-cli |
| **First case study** | `examples/omega-lock-audit.md` |
| **Korean version** | [methodology_KR.md](methodology_KR.md) |
| **License** | Apache License 2.0 |
| **How to cite** | See [README §Citing this work](../README.md#citing-this-work) |

---

## Abstract

The **antemortem** is a tactical pre-implementation reconnaissance discipline for software changes. It puts a planned change under stress *on paper* before any code is written: an LLM is given the spec and the implicated files, hypothesized risks are classified as `REAL` / `GHOST` / `NEW` / `UNRESOLVED` with primary-source `file:line` citations, and the spec is revised *before* implementation begins. The discipline differs from Gary Klein's pre-mortem (HBR 2007) in scope (a single code change, not a project) and granularity (15–30 minutes against actual source code, not strategic team brainstorming). This document specifies the seven-step protocol, the classification taxonomy, and the case-study evidence supporting it.

---

## Thesis

A postmortem is what you write after something breaks. An **antemortem** is what you do before you build. You put the planned change under stress *on paper*, use an LLM to read the existing code thoroughly, enumerate traps, classify each as real or ghost with primary-source evidence, and then revise your risk and your spec before writing a single line.

It is not a replacement for testing. It is a replacement for the first half-day of implementation that usually gets burned discovering that one of your "risks" was imaginary and one you never imagined was load-bearing.

## Why it matters now

LLM capability has made two things cheap that used to be expensive:

1. **Reading a codebase.** A capable model can scan a dozen files, trace call patterns, and correctly identify who calls what in single-digit minutes.
2. **Enumerating failure modes.** The model can list plausible traps faster and more thoroughly than a human doing it alone.

What stayed expensive: **writing correct code**. So if AI can cheaply front-load comprehension and risk enumeration, the rational move is to do that before the expensive phase, not during it.

This is the same logic driving surrogate models in expensive-evaluation optimization: put a cheap analysis between the hypothesis and the costly experiment.

## Antemortem vs pre-mortem

Gary Klein introduced the **pre-mortem** in *HBR* (2007) as a team exercise: imagine the project has failed, brainstorm why, use the list to prioritize mitigation. It is *strategic* — about the project, about decisions, about people and plans.

The **antemortem** as defined in this repository is *tactical* — about a single code change, scoped to a specific set of files, discharged in 15–30 minutes with an LLM as the reading engine. It borrows the "before the fact" temporal framing but operates on source code rather than project dynamics.

Both practices are valid and complementary. A pre-mortem asks *should we do this?* An antemortem assumes the answer is yes and asks *what does the code already tell us about the risks of doing it this way?*

In a well-run project you might do both at different scales: a pre-mortem at project kickoff, an antemortem before each non-trivial code change.

## The protocol

Seven steps. Each one short. The output is a doc.

1. **State the change.** One paragraph. What are you planning to add, remove, or refactor? What problem does it solve? What's the user-visible effect?

2. **Enumerate traps.** List every hypothesized risk. Be generous. Include ones that seem obvious. Label each: "trap" (think it's real), "worry" (unsure), or "unknown" (you haven't thought about this region yet).

3. **Hand the plan + the repo to the LLM.** Ask it to read the files your change will touch. Ask it to verify each trap against the actual code.

4. **Classify findings with primary-source citations.** For each trap:
   - **Real:** the code confirms the risk. Cite file + line. Keep on the list.
   - **Ghost:** the code shows the risk doesn't apply (e.g., the function you feared didn't recurse, the method you worried about doesn't loop). Cite file + line. Remove from the list, but keep the entry — it's load-bearing evidence of what DID NOT happen.
   - **New:** something the LLM surfaced that you hadn't thought of. Add it. Classify.

5. **Revise probabilities.** If you were tracking an overall P(success), update it with the new trap counts. The numbers are coarse but they force you to own the implication of the recon.

6. **Update the spec.** If the recon revealed that a field or a control flow is needed differently than the original spec assumed, change the spec NOW. Before the spec feels load-bearing.

7. **Save the doc.** Check it in or squirrel it away. The value is not just the recon; it is that future-you can reread it when the implementation surprises you and see which assumption broke.

Do not skip step 4. The temptation is to accept the LLM's classification without citations. The citation is the discipline. Without it you've just traded one form of hand-waving for another.

## Enhanced protocol (optional)

The seven-step protocol is sufficient for most changes. For high-stakes changes (prod deploy, data migration, security boundary) or when false-positive control matters, four additions sharpen the signal. The enhanced template at `templates/antemortem-template-enhanced.md` applies them in one integrated form.

These are not a separate protocol. They are richer forms of steps 2, 4, and 6.

### 1. Calibration dimensions (richer step 2)

Every hypothesized trap gets four axes before the recon, not just P:

- **P(issue)** — gut probability this is a real risk.
- **Evidence strength** — low/mid/high. `low` = "just a feeling", `mid` = "I've seen this pattern before", `high` = "the code structure already hints at it".
- **Blast radius** — local / module / service / system. Where damage stops if the trap fires.
- **Reversibility** — easy / hard / irrecoverable. What it costs to undo.

Why four axes: P alone underweights consequence. A 10% trap with irrecoverable blast deserves more care than a 40% trap that's local-and-easy. Scoring each axis forces weighing risks by consequence, not just likelihood.

### 2. Fine-grained classification (richer step 4)

REAL / GHOST / NEW are useful starting labels but lossy. Subdivide:

- **REAL-structural** — code confirms (static evidence).
- **REAL-runtime-uncertain** — code suggests; only runtime can confirm.
- **GHOST-mitigated** — exists but already handled upstream.
- **GHOST-unreachable** — code path doesn't actually trigger.
- **GHOST-assumption-error** — hypothesis was based on wrong mental model.
- **GHOST-test-covered** — a test explicitly pins the invariant.
- **NEW-spec-gap** — missing requirement not covered by the spec.
- **NEW-coupling** — hidden cross-module dependency.
- **NEW-operational** — ops / observability / rollout.
- **NEW-policy** — compliance / permission / legal.

Why this matters: different subtypes have different half-lives. A `GHOST-mitigated` stays mitigated only while the upstream code doesn't drift. A `GHOST-assumption-error` means *you* were wrong — the same mental-model gap may lurk elsewhere.

### 3. Explicit skeptic pass (new step 4b)

After classifying each trap, explicitly try to downgrade each REAL and NEW finding. Search for counterevidence. If you can't find any after looking, the classification stands with higher confidence.

This catches LLM confabulation. A model that produced a plausible-sounding REAL will, when pressed with "find evidence this is NOT real", sometimes downgrade it on its own. The skeptic pass forces that reconsideration to be explicit rather than implicit.

Good counter-hypothesis patterns:

- *"A mitigation might already exist in the call site / caller / config."*
- *"The code path might not actually trigger under the conditions of this change."*
- *"A test might already pin the invariant I'm worried about."*
- *"The hypothesis might be based on a wrong mental model of the module."*

The mantra still applies: *"the code shows X"* is not evidence; *"line 82 of `walk_forward.py` calls `evaluate()` once per params, no loop"* is.

### 4. Decision-first output (richer step 6)

Instead of outputting a risk list, output decisions in five blocks:

- **A. Decision blockers** — what must be resolved before coding starts (max 3).
- **B. Spec mutations required** — concrete edits to the spec.
- **C. Safe implementation path** — ordered low-risk steps.
- **D. Runtime validation needed** — what static recon cannot answer, to verify during/after implementation.
- **E. Deprioritized risks** — consciously deferred, with reason.

Why decisions, not risks: a risk list creates anxiety without obligation. A decision document pulls what you learned back into the change plan. When you reread it during implementation, each block maps to something you can act on or verify.

### When to use the enhanced form

Use the enhanced template when **any** of:

- The change touches production data, external users, or a security boundary.
- The change is hard to reverse (migrations, data deletion, published APIs).
- You are building a personal track record of predictions vs reality (section 8 of the enhanced template has a feedback table).
- You want the skeptic pass to kill false positives explicitly.

The basic template is enough for small, reversible, low-blast changes — prototypes, internal tooling, refactors that tests already cover. Don't use the enhanced form for everything; its value is in the cases where the cost of a missed risk justifies the extra minutes.

## Rubric: did the antemortem work?

A good antemortem gets three things right:

1. **Ghost detection.** Did it correctly reclassify a hypothesized trap as non-existent with primary-source evidence?
2. **Risk recalibration.** Did it change your plan of action in a way that saves time or prevents a late blocker?
3. **Spec tightening.** Did it cause the spec to change before implementation?

If an antemortem produces none of these three, it was theatre. Either the recon was too shallow, or the plan was already so well understood that an antemortem was unnecessary (rare).

## Limits

**What antemortem catches:**

- Ghost traps whose disproof lives in code.
- Missing or wrong spec fields.
- Hidden but visible-from-source mitigations.
- Places where the call pattern differs from your mental model.

**What it does not catch:**

- Runtime-only issues (race conditions, GC timing, network behavior).
- Platform-specific issues the model has no file-level evidence for.
- Product-level wrong-turns (the spec might be internally consistent and still be the wrong product).
- Things that require empirical measurement (performance claims, numerical stability).
- Your own blind spots if you only ask the model about what you already worry about.

Antemortem is a **reading discipline**. The world it knows is the world in the files. Anything outside that, it will speak about with misplaced confidence if you let it.

## Where it fits

In the AI-era development stack, antemortem sits at the front:

```
  plan -> [ANTEMORTEM] -> implement -> test -> ship -> postmortem
           ^                                            ^
           recon before building                        recon after breaking
```

Both ends use the same skill: a capable model reads code and answers structural questions. The front end is cheaper because it happens before any breakage. The back end is expensive because breakage has already happened. If you're going to invest in one, the front is higher-leverage.

This also fits the broader pattern in AI-assisted design-space optimization: when generation is cheap and evaluation is expensive, inject a cheap analysis layer between them. Antemortem is that layer for code changes.

## Practical notes

**Choosing the LLM.** Any capable reasoning model works. The first case study in this repo used Claude inside Claude Code; the companion `antemortem-cli` tool pins a specific Claude version for reproducibility. Capability matters more than brand — the model has to follow multi-file call chains and hold the change-plan in context while verifying.

**Scope the recon.** Ask the model to read the *files the change will touch*, not the whole repo. Too wide a read dilutes attention and produces vague answers. Err on the side of narrow: add files only if the first pass produces findings that clearly depend on code you didn't include.

**Insist on file + line citations.** Every trap classification should point to a specific location in the code. "The code shows X" is not evidence; "line 82 of walk_forward.py calls evaluate() once per params, no loop" is.

**Keep the traps list short enough to finish.** Seven to ten planned risks is a healthy range for a single-pass antemortem. Beyond that, split into two antemortems by component.

**Budget the recon.** Fifteen to thirty minutes for an antemortem on a change that will take 1–3 days to implement. If the recon starts sprawling past an hour, your change is too big — split it.

## When NOT to antemortem

- The change is trivial (renaming a variable, updating a docstring).
- You've been in this codebase for months and already know the answers.
- The build time is shorter than the recon time.
- You have no spec yet — write the spec first, then antemortem it.

## Reading case studies

See `examples/`. Each case study follows a fixed structure:

- Context (what the project is)
- Planned change
- Traps hypothesized (pre-recon)
- Recon protocol (what was done)
- Findings (classification with citations)
- Probability revision
- Spec changes triggered
- Implementation results

Submissions welcome via PR. Primary-source citations are the bar.
