# Antemortem — <change name>

**Date:** YYYY-MM-DD
**Author:** <you>
**Repo / branch:** <where this change will live>
**Model used for recon:** <e.g., Claude>

---

## 1. The change

<One paragraph. What are you planning to add, remove, or refactor? What problem does it solve? What is the user-visible effect? Be concrete.>

## 2. Traps hypothesized (pre-recon)

| # | trap | label | P(issue) | notes |
|---|------|-------|----------|-------|
| 1 | <description> | trap / worry / unknown | % | <why you suspect this> |
| 2 | | | | |
| 3 | | | | |

> Label guide: **trap** = I think this is real. **worry** = I'm unsure. **unknown** = I haven't thought about this region yet.

## 3. Recon protocol

- **Files handed to the model:**
  - `<path/to/file1>`
  - `<path/to/file2>`
  - `<path/to/file3>`
- **Time spent:** <minutes>
- **Scope:** <narrow / normal / wide — how much of the code the model read>

## 4. Findings (classification with citations)

For each trap in the table above, classify REAL / GHOST / NEW and cite file + line.

### Trap #1 → <REAL | GHOST | NEW>

- **Evidence:** `<file>:<line>` — <what the code shows>
- **Classification rationale:** <why the evidence supports this label>
- **Revised P(issue):** <%>

### Trap #2 → <REAL | GHOST | NEW>

- **Evidence:** `<file>:<line>`
- **Classification rationale:**
- **Revised P(issue):**

### New findings surfaced by the recon

- <anything the model pointed out that was not on the original traps list>

## 5. Probability revision

- **Pre-recon overall P(success):** <%>
- **Post-recon overall P(success):** <%>
- **What the recon bought:** <sentence summarizing where the probability moved and why>

## 6. Spec changes triggered

- <concrete edit 1 to the spec, with rationale>
- <concrete edit 2>
- <concrete edit 3>

## 7. Implementation checklist (post-recon)

- [ ] <step 1>
- [ ] <step 2>
- [ ] <step 3>

## 8. Post-implementation note (optional — fill in later)

Once implementation is done, add a short paragraph:

- Did the traps you kept on the list actually bite? Which ones?
- Did anything break that the antemortem missed?
- What would you change in the recon protocol for next time?

---

*Template version 0.1. Based on the methodology at [github.com/hibou04-ops/Antemortem](https://github.com/hibou04-ops/Antemortem).*
