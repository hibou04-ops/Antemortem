# Contributing

Two paths are open for v0.1: case studies and methodology revisions.

## Case studies

If you run an antemortem on your own codebase and want to share it, open a PR adding a file under `examples/`.

**Bar for inclusion:**

- Follow the structure of [`examples/omega-lock-audit.md`](examples/omega-lock-audit.md) (the eight sections of `templates/antemortem-template.md`).
- **Primary-source citations.** Every trap classification points to a specific file and line in the code you ran recon on. "The code shows X" is not evidence; "line 82 of `walk_forward.py` calls `evaluate()` once per params, no loop" is. This is the load-bearing discipline — PRs without citations will be asked to revise.
- Post-implementation notes welcome, including honest misses. The methodology doc explicitly lists what antemortem will not catch (runtime issues, platform quirks, product-level wrong turns). Real misses make the example more useful than polished success stories.
- Link to the repo you ran recon on if it's public. Private-repo case studies are acceptable if the cited lines are quoted inline with enough context to verify the logic.

## Methodology revisions

PRs against [`docs/methodology.md`](docs/methodology.md) are welcome. Especially useful:

- Failure modes the current doc misses.
- Protocol revisions that make step 4 (classification with citations) harder to skip.
- Additions to the "when NOT to antemortem" list based on real experience.

## Process

1. Fork.
2. Branch — `case-study/<short-name>` or `methodology/<short-description>`.
3. PR against `main`. One-line summary in the title; rationale in the description.
4. One maintainer review. Citations are verified before merge.

Case study PRs that pass the citation bar are merged without debate on conclusions — the point of antemortem is that evidence speaks. Methodology revisions go through more discussion.

## What's out of scope for v0.1

- Python CLI work (planned for v0.2).
- Schema / diffing tooling (speculative v0.3).
- Meta discussion about the word "antemortem" vs. alternatives — the name is the name for v0.1.

## Code of Conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
