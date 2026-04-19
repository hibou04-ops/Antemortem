# Antemortem case study — `omega_lock.audit`

**Date:** 2026-04-18
**Repo:** [`omega-lock`](https://github.com/hibou04-ops/omega-lock) (Python, calibration framework on PyPI)
**Model used for recon:** Claude Opus 4.7 in Claude Code
**Recon time:** ≈15 minutes

---

## 1. The change

Add a method-agnostic audit surface (`omega_lock.audit`) to the existing `omega-lock` calibration framework. The core type is an `AuditingTarget` decorator that wraps any `CalibrableTarget` (the project's existing protocol consisting of `param_space()` + `evaluate()`). Every `evaluate()` call becomes one row of an append-only trail with phase / role / round context. Declarative hard constraints are evaluated on every call. The resulting `AuditReport` serializes cleanly to JSON.

User-visible effect: anyone running a calibration — grid search, TPE, random, Bayesian, custom — can now wrap their target with four lines of code and get a reviewable, constraint-aware trail for the run.

## 2. Traps hypothesized (pre-recon)

| # | trap | label | P(issue) | notes |
|---|------|-------|----------|-------|
| 1 | Stress-measurement calls pollute the main trail | worry | 5% | minor tagging problem |
| 2 | Walk-forward fold calls inflate trail with duplicates per params | trap | 15% | "I assumed it folds internally" |
| 3 | TPE retries produce duplicate params | worry | 5% | Optuna should avoid this but edge cases exist |
| 4 | JSON serialization of `EvalResult` breaks on numpy / nested dataclasses | trap | 40% | "this always takes longer than planned" |
| 5 | Memory blow-up from storing full `EvalResult` in trail over long runs | trap | 30% | long benchmark runs could OOM |
| 6 | `run_p1_iterative` round tagging is messy | worry | 20% | multi-round coordinate descent layers |
| 7 | Windows filesystem quirks on JSON read/write | unknown | 20% | session has hit these before |

## 3. Recon protocol

- **Files handed to the model:**
  - `src/omega_lock/target.py`
  - `src/omega_lock/walk_forward.py`
  - `src/omega_lock/orchestrator.py`
  - `src/omega_lock/grid.py`
  - `src/omega_lock/p2_tpe.py`
  - `src/omega_lock/stress.py`
- **Time spent:** ≈15 min (parallel reads, then synthesis)
- **Scope:** narrow — only the files the audit change would touch. Tests and examples deliberately excluded from the first pass.

## 4. Findings

### Trap #1 (stress phase) → REAL (low risk)

- **Evidence:** `stress.py:47–143` confirms `measure_stress()` calls `target.evaluate()` once per parameter (twice for non-bool params). With the `AuditingTarget` wrapper, these calls land in the trail unless tagged differently.
- **Classification rationale:** Real but trivially handled by adding a `set_phase("stress")` call at orchestrator boundaries. Two added lines per orchestrator.
- **Revised P(issue):** 5% (unchanged — was already low).

### Trap #2 (walk-forward duplication) → **GHOST**

- **Evidence:** `walk_forward.py:82–88`:
  ```python
  for gp in top:
      r = self.test_target.evaluate(gp.params)   # one call per top-N params
  test_best = self.test_target.evaluate(best_on_train.params)  # one more
  ```
  The loop calls `test_target.evaluate(params)` exactly `top_n + 1` times. No internal folding. The module docstring at line 8–9 makes this explicit: *"The split itself is target-specific (time-series slice, fold index, etc.) and is NOT part of this module — the caller constructs the two targets."*
- **Classification rationale:** The trap assumed `WalkForward` folded internally, requiring post-hoc grouping of duplicate trail entries per params. The code shows each `evaluate()` call is one clean event. No grouping logic needed.
- **Revised P(issue):** 5% (down from 15%). Saved ≈0.5 engineer-day on logic that never got written.

### Trap #3 (TPE retries) → REAL (low risk)

- **Evidence:** `p2_tpe.py:302–337` objective function runs exactly `n_trials` times, one `target.evaluate()` per trial, `trial.number` monotonic. No pruning hooks wired, no retry on exception.
- **Classification rationale:** Any duplicate params would be legitimate TPE samples. Recording them in the trail is correct audit behavior.
- **Revised P(issue):** 5% (unchanged).

### Trap #4 (JSON serialization) → REAL but upstream-mitigated

- **Evidence:** `orchestrator.py:101` defines `_json_fallback(o)` handling sets/frozensets with stringify-fallback for unknown types. `orchestrator.py:107` defines `_eval_to_dict(r)` that explicitly drops `EvalResult.artifacts` and copies `metadata` shallowly. Same pattern duplicated in `p2_tpe.py:118`.
- **Classification rationale:** The serialization problem is already solved in the codebase. The audit module reuses these helpers instead of reinventing them.
- **Revised P(issue):** 15% (down from 40%). Residual risk: nested numpy in `metadata` stringifies lossy, but doesn't crash.

### Trap #5 (memory blow-up) → REAL but design-mitigated

- **Evidence:** `target.py:54–66` defines `EvalResult` with `metadata: dict[str, Any]` (small diagnostics) and `artifacts: dict[str, Any]` (explicitly labeled in the docstring as "large/binary objects"). The split exists precisely as a memory-safety mechanism.
- **Classification rationale:** The audit trail should store `metadata` only by default; `artifacts` opt-in via a `retain_artifacts` flag. One boolean added to the `AuditingTarget` constructor.
- **Revised P(issue):** 15% (down from 30%).

### Trap #6 (iterative rounds) → REAL but additive

- **Evidence:** `orchestrator.py:490–541` (`run_p1_iterative`) is a for-loop over `run_p1`. If the `AuditingTarget` wrapper is constructed outside the loop and passed into `run_p1_iterative`, the trail accumulates across all rounds naturally — provided the wrapper exposes `set_round(r)` to tag each round.
- **Classification rationale:** Handled by one additional field on `AuditedRun` (`round_index: int = 0`) and one method on `AuditingTarget` (`set_round(r)`). Additive change, no restructuring.
- **Revised P(issue):** 10% (down from 20%).

### Trap #7 (Windows paths) → unchanged

- **Evidence:** No new data from the recon. This is a runtime issue, not visible in files.
- **Classification rationale:** Deferred to implementation-time testing.
- **Revised P(issue):** 20% (unchanged).

### New findings (surfaced by the recon)

- **Multi-target audit is mandatory.** `run_p1` uses up to three separate `CalibrableTarget` instances simultaneously: `train_target`, `test_target`, `holdout_target`. The original spec had a `phase` field but no role field, meaning the trail couldn't distinguish which target produced which row.
  - **Fix triggered:** Added `target_role: str` to `AuditedRun`. Added `shared_trail: list | None` and `shared_counter: count | None` parameters to `AuditingTarget.__init__` so multiple wrappers can contribute to one ordered trail.

## 5. Probability revision

- **Pre-recon P(full spec ships on time):** 55–65%
- **Post-recon P(full spec ships on time):** 70–78%
- **What the recon bought:** Trap #2 reclassified as ghost (no grouping logic to write). Traps #4, #5, #6 each downgraded by existing-pattern discoveries. One new requirement (target_role) surfaced before implementation, avoiding a late-stage retrofit.

## 6. Spec changes triggered

- Add `phase`, `call_index`, `target_role`, `round_index` fields to `AuditedRun`.
- Add `shared_trail` and `shared_counter` parameters to `AuditingTarget.__init__` to enable multi-target audits with one global trail.
- Add `retain_artifacts: bool = False` default to keep the trail memory-safe.
- Reuse `_json_fallback` from existing code. Do NOT reinvent a serializer.

## 7. Implementation checklist (post-recon)

- [x] `src/omega_lock/audit/_types.py` — Constraint, AuditedRun, AuditReport
- [x] `src/omega_lock/audit/_target.py` — AuditingTarget decorator with set_phase / set_round
- [x] `src/omega_lock/audit/_scorecard.py` — text renderer
- [x] `src/omega_lock/audit/__init__.py` — public API + make_report
- [x] `tests/test_audit.py` — 20 tests covering protocol, shared trail, JSON roundtrip, scorecard, GridSearch integration

## 8. Post-implementation note

Implementation took approximately one engineer-day. Result:

- **20 new tests, 20 passing on first run.**
- **Full suite: 169/169 (was 149).** No regressions.
- **End-to-end smoke test on a `PhantomKeyhole` target:** 36 evaluate calls captured cleanly across train + test target roles, 3 constraints evaluated, scorecard rendered, JSON roundtrip verified.

What bit that the antemortem correctly flagged:

- The `target_role` field surfaced as "new finding" was genuinely necessary. If it had been retrofitted during implementation, every `AuditedRun` construction site would have needed editing after the fact.

What the antemortem didn't catch:

- Windows cp949 terminal encoding rejected em-dashes in the scorecard string output. Had to replace `—` with `--`. This is exactly the runtime-only issue the methodology doc warns antemortem will not catch. Caught by the end-to-end smoke test and fixed in minutes, but noted here as honest tracking of a miss.

What would change next time:

- Add a "string literal audit" trap category — scan output strings for characters that might fail in non-UTF-8 terminals. This is pattern-matchable from code and would have caught the cp949 issue from file-level evidence alone.
