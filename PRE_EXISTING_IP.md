# Pre-existing Intellectual Property Declaration

> **Purpose**: This document is a tamper-evident timestamped declaration that the
> work in this repository constitutes pre-existing personal intellectual property
> of the Primary Author, authored prior to and independent of any current or
> future employment relationship.

## Repository Identification

- **Repository**: [hibou04-ops/Antemortem](https://github.com/hibou04-ops/Antemortem)
- **License**: Apache License 2.0
- **Primary Author**: **Kyunghoon Gwak (곽경훈)** — operating as [@hibou04-ops](https://github.com/hibou04-ops)
  - Primary contact email: `hibouaile04@gmail.com`
  - Git author email on commits prior to 2026-04-28: `hibou04@gmail.com`
  - Both email addresses are verified personal accounts of the Primary Author

## Authorship Timeline (Tamper-Evident)

The following git artifacts establish the authorship timeline. The git commit graph
and the public GitHub remote (`github.com/hibou04-ops/Antemortem`) provide
independent timestamp witnesses.

| Anchor | Commit Hash | Date (KST) | Description |
|---|---|---|---|
| First commit | `7e378ad7d38d4a3a837f43056d78d805822df3fc` | 2026-04-20 03:41:38 +0900 | Antemortem v0.1 — methodology, template, first case study |
| v0.1.1 release | (see CHANGELOG.md) | 2026-04-20 | Enhanced protocol + adjacent-practices + FAQ |
| Apache 2.0 relicense | `8bf43b4` | 2026-04-22 | MIT → Apache 2.0 for patent grant + trademark preservation |
| Pre-employment snapshot | (tagged on commit) | 2026-04-28 | This declaration committed; tagged `pre-employment-ip-snapshot-2026-04-28` |

## Scope of Pre-existing IP

The following work product is declared as pre-existing personal intellectual property:

1. **Methodology**: The Antemortem discipline — seven-step protocol, risk
   classification taxonomy (`REAL` / `GHOST` / `NEW` / `UNRESOLVED`), citation
   discipline, and the framing of pre-implementation reconnaissance.
2. **Templates**: All materials under `templates/`.
3. **Case Studies**: All materials under `examples/`.
4. **Documentation**: README, README_KR, EASY_README, EASY_README_KR, CHANGELOG,
   CONTRIBUTING, methodology papers under `docs/`.
5. **Specific Terminology and Application**: The specific *application* of
   the term "Antemortem" to label a software-engineering pre-implementation
   reconnaissance discipline, as a distinct usage within this methodology
   corpus. The companion tool naming "antemortem-cli". The risk-classification
   taxonomy `REAL` / `GHOST` / `NEW` / `UNRESOLVED` as specific terminology
   defined and used in this methodology. *No claim is made to the generic
   English word "antemortem" (meaning "before death") or to standard software
   engineering terminology not originating in this corpus; the claim is to
   the specific application, definitions, and arrangement of these terms
   within this methodology.*

## Development Conditions

This work was developed:

- Using **personal time** (outside of any third-party working hours)
- Using **personal equipment** (no employer-issued hardware)
- Using **personal accounts** (no employer-issued cloud, LLM, or API credentials)
- **Without reference** to any third party's confidential or proprietary information

## Use in Future Employment Agreements

This declaration is intended to be attached as a Schedule / Exhibit (commonly
"Schedule A: Pre-existing IP") to any future employment, contractor, or
consulting agreement, to clarify that:

- The work in this repository remains the personal property of the Primary Author.
- Future development on this codebase, conducted on personal time and outside the
  scope of any employment, continues to be the Primary Author's personal IP.
- Any contributions from a future employer's domain, made on employer time using
  employer resources, would be governed by the relevant employment agreement —
  the boundary is preserved by maintaining a separate repository, fork, or
  branch for any such employer-domain contributions.
- Apache 2.0 license terms continue to apply, including the patent grant.

## Verification

To independently verify this declaration:

1. Inspect git log:
   ```
   git log --format="%H | %ai | %an <%ae>" | grep "Hibou04-ops"
   ```
2. Confirm tag (when committed):
   ```
   git tag -l "pre-employment-ip-snapshot-*"
   git show pre-employment-ip-snapshot-2026-04-28
   ```
3. Cross-reference with public GitHub timestamps:
   - https://github.com/hibou04-ops/Antemortem/commit/7e378ad7d38d4a3a837f43056d78d805822df3fc
   - https://github.com/hibou04-ops/Antemortem/releases

---

**Declaration date**: 2026-04-28
**License**: Apache License 2.0
**Document version**: 1.0
