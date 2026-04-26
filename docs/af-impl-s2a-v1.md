# af-impl-s2a-v1
# Project: AgentForge
# Document Type: Sprint 2a Implementation Specification
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

Sprint 2a delivers the Attribution Engine ONLY.
The fix engine is NOT built in this sprint.

This is non-negotiable. Attribution must be manually validated (Gate 3)
before the fix engine is built. Building them together risks optimising the wrong thing.

---

## PREREQUISITE

Sprint 1 must be complete and validated on a real agent before Sprint 2a begins.
Gate 1: APPROVED
Gate 2: Required before disk write
Gate 3: Must be completed at END of Sprint 2a before Sprint 2b begins

---

## SPRINT 2a SCOPE

**In scope:**
- Perturbation-based attribution engine
- Attribution LLM assist (EP-004)
- FailureTrace persistence
- Attribution human review interface
- Gate 3 enforcement

**Out of scope:**
- Fix engine (Sprint 2b)
- Layer 2, 2.5, 3 (Sprint 2c)
- SDK (Sprint 3)

---

## TASK LIST

### T-2a-001 — Attribution Engine Core
**File:** backend/engine/attribution/engine.py
**Maps to:** FR-009

**Must implement:**
- receive_attribution_request(eval_run_id) — loads failing run, prompt version, test suite
- fetch_failing_criteria(eval_run_id) — returns list of failed criteria with scores
- run_perturbation_loop(prompt_version, failing_criteria) — main attribution logic
- persist_attribution_results(eval_run_id, traces) — saves FailureTrace records

**Functions must not exceed 20 lines each.**

---

### T-2a-002 — Perturbation Runner
**File:** backend/engine/attribution/perturbation.py
**Maps to:** FR-009

**Must implement:**
- neutralise_module(prompt_version, module_id) — returns copy with module blanked
- run_eval_without_module(neutralised_version, test_suite, llm_client) — re-runs eval
- compute_score_delta(original_score, ablated_score) — returns delta (positive = module helped)
- rank_modules_by_impact(deltas) — sorts modules by negative delta (most blame first)

**Key rule:** Module with highest negative delta = primary failure contributor.
A module that when removed causes score to IMPROVE was hurting the eval.

---

### T-2a-003 — Attribution Scorer
**File:** backend/engine/attribution/scorer.py
**Maps to:** FR-009

**Must implement:**
- compute_confidence(delta, baseline_score) — confidence = abs(delta) / baseline_score
- build_failure_trace(module, delta, confidence, failure_types) — returns FailureTrace data
- classify_failure_types(failing_criteria) — maps criteria failures to failure type enum

---

### T-2a-004 — Attribution LLM Assist
**File:** backend/engine/attribution/llm_assist.py
**Maps to:** FR-009, EP-004

**Must implement:**
- get_attribution_hints(modules, failures, llm_client) — calls EP-004 prompt
- parse_attribution_response(llm_response) — extracts module_impact list
- merge_hints_with_perturbation(perturbation_results, hints) — combines both signals

**Note:** LLM assist is supplementary. Perturbation result is authoritative.
If LLM hint and perturbation disagree, perturbation wins.

---

### T-2a-005 — FailureTrace ORM Model Update
**File:** backend/models/failure_trace.py
**Maps to:** af-data-contract-spec-v1

**Must implement:**
```python
class FailureTrace(Base):
    id, eval_run_id, module_id, module_tag,
    contribution_score, confidence, failure_types (ARRAY),
    explanation, llm_hint_agreement (boolean),
    created_at
```

---

### T-2a-006 — Attribution API Endpoints
**File:** backend/api/attribution.py
**Maps to:** FR-009, FR-024

**Endpoints:**
- POST /evals/{run_id}/attribute — trigger attribution (async)
- GET  /evals/{run_id}/attribution — get attribution results
- POST /evals/{run_id}/attribution/validate — human marks attribution as correct/incorrect
- GET  /attribution/gate-status — check Gate 3 status

---

### T-2a-007 — Gate 3 Enforcement
**File:** backend/core/gates.py
**Maps to:** FR-024

**Must implement:**
- check_gate3_status() — returns Gate3Status(satisfied, validated_count, correct_count)
- record_attribution_validation(run_id, verdict) — logs human validation result
- Gate 3 satisfied when: validated_count >= 3 AND correct_count >= 2

**Integration:** Fix engine endpoints (Sprint 2b) must call check_gate3_status()
and return 423 Locked if not satisfied.

---

## GATE 3 — END OF SPRINT 2a

Gate 3 is satisfied when ALL of the following are true:

- [ ] Attribution engine runs successfully on at least 3 real failing eval runs
- [ ] Human reviewer marks attribution as correct in at least 2 of 3 runs
- [ ] Attribution confidence scores are above 0.7 for correct attributions
- [ ] check_gate3_status() returns satisfied: true
- [ ] Human reviewer signs off in writing

**Gate 3 approver signature:** _______________
**Date:** _______________

Only after Gate 3 is satisfied may Sprint 2b (fix engine) begin.

---

## EXIT CRITERIA FOR SPRINT 2a

- [ ] T-2a-001 through T-2a-007 complete
- [ ] Attribution runs end-to-end on a real failing eval
- [ ] FailureTrace records persisted correctly
- [ ] Gate 3 satisfied and signed off
- [ ] All functions have docstrings and are under 20 lines
- [ ] Gate 2 approval obtained before disk write
