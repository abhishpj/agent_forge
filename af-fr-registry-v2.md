# af-fr-registry-v1
# Project: AgentForge
# Document Type: Functional Requirements Registry
# Version: 2.0
# Status: Draft
# Last Updated: 2026-04
# Change: v2.0 — added FR-018 to FR-025 (mandatory additions + criteria template layer)

---

## PURPOSE

This registry defines all functional requirements for AgentForge.
Every feature built must trace back to a requirement in this file.
No code is written for functionality not listed here.

---

## REQUIREMENT INDEX

| ID     | Name                              | Priority | Sprint | Status  |
|--------|-----------------------------------|----------|--------|---------|
| FR-001 | Agent Registration                | P0       | S1     | Draft   |
| FR-002 | Prompt Module Management          | P0       | S1     | Draft   |
| FR-003 | Test Suite Definition             | P0       | S1     | Draft   |
| FR-004 | Evaluation Run Execution          | P0       | S1     | Draft   |
| FR-005 | Layer 1 Deterministic Validation  | P0       | S1     | Draft   |
| FR-006 | Layer 2 Statistical Validation    | P1       | S2c    | Draft   |
| FR-007 | Layer 2.5 Reasoning Signals       | P1       | S2c    | Draft   |
| FR-008 | Layer 3 LLM Judge                 | P1       | S2c    | Draft   |
| FR-009 | Failure Attribution Engine        | P0       | S2a    | Draft   |
| FR-010 | Auto Fix Generation               | P0       | S2b    | Draft   |
| FR-011 | Prompt Version Management         | P0       | S1     | Draft   |
| FR-012 | Deployment Gate                   | P0       | S1     | Draft   |
| FR-013 | Python SDK                        | P1       | S3     | Draft   |
| FR-014 | Production Call Capture           | P1       | S3     | Draft   |
| FR-015 | Drift Detection                   | P1       | S4     | Draft   |
| FR-016 | Rollback Control                  | P1       | S4     | Draft   |
| FR-017 | Control Dashboard                 | P1       | S4     | Draft   |
| FR-018 | Trajectory Validator              | P0       | S1     | Draft   |
| FR-019 | Tool Selection Precision Metric   | P0       | S1     | Draft   |
| FR-020 | Operating Envelope Validators     | P0       | S1     | Draft   |
| FR-021 | Faithfulness Constraint in Judge  | P0       | S1     | Draft   |
| FR-022 | Criteria Template Library         | P0       | S1     | Draft   |
| FR-023 | Domain Starter Packs              | P0       | S1     | Draft   |
| FR-024 | Attribution Validation Gate       | P0       | S2a    | Draft   |
| FR-025 | Pairwise Fix Comparison           | P1       | S2b    | Draft   |

---

## REQUIREMENTS

### FR-001 through FR-017
See af-fr-registry-v1.md version 1.0 — unchanged.

---

### FR-018 — Trajectory Validator

**Description:**
The system must validate that an agent's tool call sequence matches the expected trajectory,
not just the final output.

**Acceptance Criteria:**
- User defines expected tool call sequence in test case config: ["tool_a", "tool_b", "tool_c"]
- Agent output includes a tool_calls array in the response or trace
- Validator checks: correct tools called, correct order, no extra tools, no missing tools
- Partial match scoring: each correct step contributes proportionally to score
- Returns: score (0.0-1.0), matched steps, missing steps, unexpected steps
- Runs as Layer 1 (deterministic) — no LLM required
- If agent does not use tools, this validator is skipped gracefully

**Dependencies:** FR-004, FR-005
**Priority:** P0
**Sprint:** 1

---

### FR-019 — Tool Selection Precision Metric

**Description:**
The system must measure whether the agent selected the correct tool for each reasoning step,
independent of whether the overall trajectory was correct.

**Acceptance Criteria:**
- User defines expected tool for each step in test case config
- Precision = correct tool selections / total tool selections
- Recall = correct tool selections / total expected tool selections
- F1 score computed from precision and recall
- Results returned as structured score with per-step breakdown
- Runs as Layer 1 (deterministic)
- Works independently of trajectory validator — can be used without ordered sequence

**Dependencies:** FR-018
**Priority:** P0
**Sprint:** 1

---

### FR-020 — Operating Envelope Validators

**Description:**
The system must support cost and latency gates that fail an eval if resource usage
exceeds configured thresholds — regardless of output quality.

**Acceptance Criteria:**
- User configures per test suite: max_cost_usd (float), max_latency_ms (integer)
- If actual cost exceeds max_cost_usd → eval result includes cost_gate: failed
- If actual latency exceeds max_latency_ms → eval result includes latency_gate: failed
- Cost and latency gates are independent of quality score
- Overall eval status is failed if any gate fails regardless of quality score
- Thresholds are optional — if not configured, gates are skipped
- Runs as Layer 1 (deterministic)

**Dependencies:** FR-004, FR-005
**Priority:** P0
**Sprint:** 1

---

### FR-021 — Faithfulness Constraint in LLM Judge

**Description:**
The LLM judge (Layer 3) must explicitly evaluate faithfulness — whether the output
contains only information supported by the provided context, with no hallucinations.

**Acceptance Criteria:**
- EP-001 (eval judge prompt) updated to include faithfulness as a mandatory criterion
- Faithfulness criterion evaluates: does output introduce facts not in input context?
- Faithfulness is scored independently (0.0-1.0) alongside other criteria
- Faithfulness failure is classified as type: hallucination in failure results
- Faithfulness criterion is always evaluated when layer 3 runs — cannot be disabled
- Faithfulness score weighted at minimum 0.3 in overall layer 3 score

**Dependencies:** FR-008
**Priority:** P0
**Sprint:** 1 (update to EP-001 prompt)

---

### FR-022 — Criteria Template Library

**Description:**
The system must provide a library of pre-built validator templates that users can
apply to their test cases without writing criteria from scratch.

**Acceptance Criteria:**
- Template library is accessible via GET /criteria-templates
- Each template has: id, name, description, validator_type, default_config, tags, domain
- Templates are filterable by domain and validator_type
- User can apply a template to a test case criterion with one API call
- Applied template populates validator_type and validator_config automatically
- Templates are read-only — users cannot modify platform templates
- Users can save custom templates from their own criteria

**Sprint 1 must include minimum 15 templates across domains**

**Dependencies:** FR-003, FR-005
**Priority:** P0
**Sprint:** 1

---

### FR-023 — Domain Starter Packs

**Description:**
The system must provide domain-specific starter packs that create a pre-populated
test suite with relevant criteria when a user creates a new agent in a known domain.

**Acceptance Criteria:**
- Starter packs available for: QA automation, RAG/knowledge base, customer support,
  code generation, data extraction, summarisation
- Each pack creates: 3-5 test cases with pre-filled criteria appropriate to the domain
- User selects domain when creating agent — starter pack is optional, not mandatory
- Starter pack test cases are fully editable after creation
- Starter packs are versioned and maintained by the platform

**Sprint 1 must include minimum 3 domain starter packs:**
- QA automation agent
- RAG / knowledge base agent
- General purpose agent

**Dependencies:** FR-022, FR-003
**Priority:** P0
**Sprint:** 1

---

### FR-024 — Attribution Validation Gate (Gate 3)

**Description:**
Before the fix engine is built or used, attribution accuracy must be validated
manually on real agent failures. This gate enforces the correct build sequence.

**Acceptance Criteria:**
- Attribution engine provides a human review interface: show attribution result + allow human to mark as correct/incorrect
- Attribution validation runs must be logged: run_id, attributed_module, human_verdict, timestamp
- Gate 3 is satisfied when: minimum 3 real agent failures validated, attribution correct in at least 2 of 3
- Gate 3 status is visible in the admin interface
- Fix engine endpoints return 423 Locked if Gate 3 is not satisfied
- Gate 3 can only be approved by a human — not automated

**Dependencies:** FR-009
**Priority:** P0
**Sprint:** 2a

---

### FR-025 — Pairwise Fix Comparison

**Description:**
When a fix variant is generated, the system must run a pairwise LLM comparison
between the original output and the fixed output — not just compare scores numerically.

**Acceptance Criteria:**
- After fix eval completes, pairwise judge runs: "which output better satisfies these criteria?"
- Pairwise judge returns: winner (original/fixed), reasoning, confidence (0.0-1.0)
- Result stored alongside score delta in FixVariant record
- Fix is accepted only if: score delta >= threshold AND pairwise judge selects fixed as winner
- If score improves but pairwise judge prefers original → human review flagged
- Pairwise comparison uses EP-001 judge with pairwise mode

**Dependencies:** FR-010
**Priority:** P1
**Sprint:** 2b
