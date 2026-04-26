# af-impl-s2b-v1
# Project: AgentForge
# Document Type: Sprint 2b Implementation Specification
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

Sprint 2b delivers the Fix Engine ONLY.
This sprint cannot begin until Gate 3 is satisfied.

If you are reading this and Gate 3 is not yet approved — STOP.
Return to Sprint 2a and complete Gate 3 first.

---

## PREREQUISITE — NON-NEGOTIABLE

Gate 3 must be APPROVED before any code in this sprint is written.
check_gate3_status() must return satisfied: true.
Fix engine endpoints return 423 Locked until Gate 3 is satisfied.

---

## SPRINT 2b SCOPE

**In scope:**
- Fix generation engine
- Fix variant persistence
- Pairwise comparison judge
- Fix verification loop
- Auto vs manual trigger mode (per agent config)

**Out of scope:**
- Attribution engine (Sprint 2a — already built)
- Layer 2, 2.5, 3 (Sprint 2c)
- SDK (Sprint 3)

---

## TASK LIST

### T-2b-001 — Fix Generator
**File:** backend/engine/fix/generator.py
**Maps to:** FR-010, EP-003

**Must implement:**
- receive_fix_request(eval_run_id) — loads attribution result for the run
- extract_failing_module(attribution_result) — gets the primary attributed module
- build_fix_prompt(module, failing_criteria, failures) — constructs EP-003 prompt
- call_fix_llm(fix_prompt, llm_client) — sends to LLM, returns raw response
- parse_fix_response(raw_response) — extracts updated_content, changes_summary
- validate_other_modules_unchanged(original_version, fixed_version) — ensures only target module changed

**Critical rule:** Fix must ONLY rewrite the attributed failing module.
All other modules must be byte-for-byte identical to the original version.

---

### T-2b-002 — Fix Version Creator
**File:** backend/engine/fix/version_creator.py
**Maps to:** FR-010, FR-011

**Must implement:**
- create_fix_version(original_version_id, fixed_module_content, module_id) — creates new PromptVersion
- copy_all_modules_except_target(original_version, target_module_id) — copies unchanged modules
- apply_fixed_module(new_version, module_id, new_content) — applies the fix
- register_fix_variant(source_run_id, original_version_id, fixed_version_id, changes) — persists FixVariant

---

### T-2b-003 — Fix Verifier
**File:** backend/engine/fix/verifier.py
**Maps to:** FR-010

**Must implement:**
- trigger_reverification_eval(fixed_version_id, test_suite_id) — runs eval on fixed version
- compute_score_delta(original_score, fixed_score) — returns delta
- check_improvement_threshold(delta, min_threshold) — True if delta >= threshold
- determine_fix_acceptance(delta, pairwise_result, threshold) — final accept/reject decision

**Acceptance rule:** Fix accepted ONLY if:
- score_delta >= config.fix_minimum_score_delta (default 0.1) AND
- pairwise_judge selects fixed as winner

If score improves but pairwise prefers original → flag for human review, do not auto-accept.

---

### T-2b-004 — Pairwise Comparison Judge
**File:** backend/engine/fix/pairwise_judge.py
**Maps to:** FR-025, EP-001

**Must implement:**
- run_pairwise_comparison(input, original_output, fixed_output, criteria, llm_client)
- build_pairwise_prompt(input, original_output, fixed_output, criteria) — constructs prompt
- parse_pairwise_response(raw_response) — extracts winner, reasoning, confidence
- Returns: PairwiseResult(winner, reasoning, confidence)

**Prompt note:** Uses EP-001 judge in pairwise mode.
Prompt must not reveal which output is "original" vs "fixed" — prevent position bias.
Outputs must be labelled "Output A" and "Output B" randomly.

---

### T-2b-005 — FixVariant ORM Model
**File:** backend/models/fix_variant.py
**Maps to:** af-data-contract-spec-v1

**Must implement:**
```python
class FixVariant(Base):
    id, source_eval_run_id, original_version_id,
    fixed_version_id, changed_module_id,
    changes_summary (ARRAY),
    original_score, fixed_score, score_delta,
    pairwise_winner, pairwise_confidence, pairwise_reasoning,
    accepted, human_review_required,
    created_at
```

---

### T-2b-006 — Fix API Endpoints
**File:** backend/api/fixes.py
**Maps to:** FR-010, FR-025

**Endpoints:**
- POST /evals/{run_id}/fix — trigger fix generation (returns 423 if Gate 3 not satisfied)
- GET  /fixes/{fix_id} — get fix variant with score comparison and pairwise result
- POST /fixes/{fix_id}/accept — manually accept fix → activates fixed version
- POST /fixes/{fix_id}/reject — manually reject fix
- GET  /agents/{agent_id}/fixes — list all fix variants for agent

---

### T-2b-007 — Auto vs Manual Trigger Config
**File:** Update Agent model and config
**Maps to:** FR-010

**Must implement:**
- Add fix_trigger_mode to Agent model: "auto" | "manual" (default: "manual")
- Auto mode: attribution completion triggers fix generation automatically
- Manual mode: user explicitly calls POST /evals/{run_id}/fix
- Auto mode still requires Gate 3 to be satisfied

---

## EXIT CRITERIA FOR SPRINT 2b

- [ ] T-2b-001 through T-2b-007 complete
- [ ] Fix generates and only modifies attributed module (other modules unchanged confirmed)
- [ ] Pairwise comparison runs and returns structured result
- [ ] Score delta and pairwise result both required for acceptance
- [ ] Human review flag triggers correctly when score/pairwise disagree
- [ ] 423 Locked returned when Gate 3 not satisfied
- [ ] All functions have docstrings and are under 20 lines
- [ ] Gate 2 approval obtained before disk write
