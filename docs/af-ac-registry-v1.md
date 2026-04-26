# af-ac-registry-v1
# Project: AgentForge
# Document Type: Acceptance Criteria Registry
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

This registry defines the acceptance criteria that must be met before AgentForge Sprint 1 is considered complete.
Each criterion maps to one or more functional requirements and must be verifiable by a human reviewer or automated test.

---

## AC INDEX

| ID     | Maps To       | Description                                      | Verifiable By  |
|--------|---------------|--------------------------------------------------|----------------|
| AC-001 | FR-001        | Agent can be created, retrieved, updated, deleted | API test       |
| AC-002 | FR-002        | Prompt modules can be managed independently      | API test       |
| AC-003 | FR-002        | Raw prompt auto-splits into tagged modules       | API test       |
| AC-004 | FR-003        | Test suite with criteria and validators created  | API test       |
| AC-005 | FR-004        | Eval run executes and returns structured results | API test       |
| AC-006 | FR-005        | Layer 1 validators all pass deterministically    | Unit test      |
| AC-007 | FR-005        | Layer 1 catches schema violation correctly       | Unit test      |
| AC-008 | FR-005        | Layer 1 catches forbidden string correctly       | Unit test      |
| AC-009 | FR-009        | Attribution identifies correct failing module    | Integration    |
| AC-010 | FR-010        | Fix variant improves score vs original           | Integration    |
| AC-011 | FR-011        | Version history preserved on every save          | API test       |
| AC-012 | FR-012        | Deployment gate blocks activation without pass   | API test       |
| AC-013 | NFR-002       | Layer 1 suite of 20 checks completes under 500ms | Performance    |
| AC-014 | NFR-004       | SDK does not break host app when backend is down | Fault test     |
| AC-015 | NFR-011       | All functions under 20 lines with docstrings     | Code review    |

---

## ACCEPTANCE CRITERIA DETAIL

### AC-001 — Agent CRUD

**Requirement:** FR-001
**Description:** A user must be able to create, retrieve, update, and delete an agent via the API.

**Test Steps:**
1. POST /agents with valid payload → returns 201 with agent ID
2. GET /agents/{id} → returns agent with correct fields
3. PUT /agents/{id} with updated name → returns 200 with updated name
4. DELETE /agents/{id} → returns 204
5. GET /agents/{id} after delete → returns 404

**Pass Criteria:** All 5 steps return expected status codes and payloads.

---

### AC-002 — Prompt Module Management

**Requirement:** FR-002
**Description:** A user must be able to add, edit, reorder, and delete prompt modules independently.

**Test Steps:**
1. POST /prompts with modules array → returns 201 with version ID
2. GET /prompts/{id}/modules → returns all modules in correct order
3. PUT /prompts/{id}/modules/{module_id} with updated content → returns 200
4. DELETE /prompts/{id}/modules/{module_id} → returns 204
5. GET /prompts/{id}/assembled → returns full prompt assembled from remaining modules

**Pass Criteria:** All 5 steps return expected status codes and payloads.

---

### AC-003 — Raw Prompt Auto-Split

**Requirement:** FR-002
**Description:** A raw unstructured prompt must be automatically split into tagged modules.

**Test Steps:**
1. POST /prompts/parse with raw prompt text → returns suggested module split
2. Verify each returned module has a valid tag (SYSTEM, CONTEXT, INSTRUCTIONS, FORMAT, EXAMPLES, GUARDRAILS)
3. Verify assembled modules reproduce original prompt content

**Pass Criteria:** All modules have valid tags; assembled text matches original content.

---

### AC-004 — Test Suite Creation

**Requirement:** FR-003
**Description:** A user must be able to create a test suite with typed criteria and validator assignments.

**Test Steps:**
1. POST /test-suites with agent ID, name, and test cases array → returns 201
2. Each test case includes input, criteria list, validator type per criterion, pass threshold
3. GET /test-suites/{id} → returns suite with all test cases and criteria intact

**Pass Criteria:** Suite created and retrieved with all fields intact.

---

### AC-005 — Eval Run Execution

**Requirement:** FR-004
**Description:** An eval run must execute against a prompt version and return structured results.

**Test Steps:**
1. POST /evals/run with prompt version ID and test suite ID → returns 202 with run ID
2. GET /evals/{run_id} → returns results with overall score, per-criterion scores, failures
3. Results include layer breakdown showing which layer each criterion was evaluated by
4. Run status transitions: pending → running → passed or failed

**Pass Criteria:** Results returned with all required fields; status transitions correctly.

---

### AC-006 — Layer 1 Deterministic Pass

**Requirement:** FR-005
**Description:** Running Layer 1 validators twice on identical inputs must produce identical results.

**Test Steps:**
1. Run schema validator on valid JSON output → pass
2. Run same validator again on same output → same pass result
3. Run schema validator on invalid JSON output → fail with reason
4. Run same validator again → same fail result

**Pass Criteria:** 100% identical results on repeated runs.

---

### AC-007 — Layer 1 Schema Violation Detection

**Requirement:** FR-005
**Description:** Schema validator must correctly detect and report schema violations.

**Test Steps:**
1. Define schema requiring fields: id (string), score (number), status (string)
2. Submit output missing `score` field → fail with reason citing missing field
3. Submit output with `score` as string instead of number → fail with type error reason
4. Submit fully valid output → pass

**Pass Criteria:** Violations detected and reported with correct field and type information.

---

### AC-008 — Layer 1 Forbidden String Detection

**Requirement:** FR-005
**Description:** Forbidden string validator must detect prohibited content in output.

**Test Steps:**
1. Configure forbidden strings: ["TODO", "FIXME", "hardcoded"]
2. Submit output containing "TODO: fix this" → fail with match location
3. Submit output containing none of the forbidden strings → pass

**Pass Criteria:** Forbidden strings detected; clean output passes.

---

### AC-009 — Failure Attribution Accuracy

**Requirement:** FR-009
**Description:** Attribution engine must identify the correct failing module in a known failure scenario.

**Test Steps:**
1. Create prompt with SYSTEM, INSTRUCTIONS, FORMAT modules
2. Deliberately corrupt FORMAT module to produce invalid output format
3. Run eval → overall fail
4. Trigger attribution → FORMAT module must be identified as primary contributor
5. Attribution confidence score must be above 0.7

**Pass Criteria:** FORMAT module identified as primary contributor with confidence above 0.7.

---

### AC-010 — Fix Generation Improvement

**Requirement:** FR-010
**Description:** Auto-generated fix variant must produce a higher eval score than the original.

**Test Steps:**
1. Use failing prompt version from AC-009
2. Trigger fix generation on identified failing module
3. Verify fix rewrites only the FORMAT module, not others
4. Run eval on fixed variant → score must be higher than original by at least 0.1
5. Side-by-side score comparison returned

**Pass Criteria:** Fixed variant score improves by minimum 0.1; other modules unchanged.

---

### AC-011 — Version History Preservation

**Requirement:** FR-011
**Description:** Every prompt save must create a new immutable version; history must never be lost.

**Test Steps:**
1. Create prompt version 1
2. Edit a module → creates version 2
3. Edit again → creates version 3
4. GET /prompts/{id}/versions → returns versions 1, 2, 3 with content intact
5. Attempt to edit version 1 directly → must be rejected (immutable)

**Pass Criteria:** All versions preserved; direct edit of prior version rejected.

---

### AC-012 — Deployment Gate Enforcement

**Requirement:** FR-012
**Description:** System must block activation of a prompt version that has not passed the deployment gate.

**Test Steps:**
1. Configure test suite as deployment gate for agent
2. Create new prompt version
3. Attempt POST /prompts/{id}/activate without passing eval → returns 422 with clear error
4. Run eval suite → fail result
5. Attempt activate again → still blocked
6. Fix prompt → run eval → pass result
7. Attempt activate → returns 200 success

**Pass Criteria:** Activation blocked until eval passes; clear error messages at each step.

---

### AC-013 — Layer 1 Performance

**Requirement:** NFR-002
**Description:** A test suite with 20 Layer 1 checks must complete within 500ms.

**Test Steps:**
1. Configure test suite with 20 Layer 1 checks (mix of schema, regex, required fields, length)
2. Run eval 5 times and record duration of Layer 1 phase
3. All 5 runs must complete Layer 1 within 500ms

**Pass Criteria:** All runs complete Layer 1 within 500ms.

---

### AC-014 — SDK Fault Tolerance

**Requirement:** NFR-004
**Description:** SDK must not break host application when AgentForge backend is unreachable.

**Test Steps:**
1. Configure SDK with unreachable backend URL
2. Call agentforge.complete() with valid model and input
3. LLM call must still complete and return result
4. No exception raised in host application
5. SDK logs silent warning internally

**Pass Criteria:** LLM call succeeds; no exception in host application.

---

### AC-015 — Code Style Compliance

**Requirement:** NFR-011
**Description:** All source files must comply with defined code style standards.

**Verification Method:** Manual code review by reviewer

**Checklist:**
- [ ] No function exceeds 20 lines
- [ ] All functions have docstrings
- [ ] No abbreviations in variable names
- [ ] No nested ternaries
- [ ] No magic numbers (all constants named)
- [ ] One responsibility per function
- [ ] No one-liners for complex logic

**Pass Criteria:** All checklist items confirmed by reviewer.
