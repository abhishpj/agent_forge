# af-fr-registry-v1
# Project: AgentForge
# Document Type: Functional Requirements Registry
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

This registry defines all functional requirements for AgentForge.
Every feature built must trace back to a requirement in this file.
No code is written for functionality not listed here.

---

## REQUIREMENT INDEX

| ID     | Name                          | Priority | Status  |
|--------|-------------------------------|----------|---------|
| FR-001 | Agent Registration            | P0       | Draft   |
| FR-002 | Prompt Module Management      | P0       | Draft   |
| FR-003 | Test Suite Definition         | P0       | Draft   |
| FR-004 | Evaluation Run Execution      | P0       | Draft   |
| FR-005 | Layer 1 Deterministic Validation | P0    | Draft   |
| FR-006 | Layer 2 Statistical Validation | P1      | Draft   |
| FR-007 | Layer 2.5 Reasoning Signals   | P1       | Draft   |
| FR-008 | Layer 3 LLM Judge             | P1       | Draft   |
| FR-009 | Failure Attribution Engine    | P0       | Draft   |
| FR-010 | Auto Fix Generation           | P0       | Draft   |
| FR-011 | Prompt Version Management     | P0       | Draft   |
| FR-012 | Deployment Gate               | P0       | Draft   |
| FR-013 | Python SDK                    | P1       | Draft   |
| FR-014 | Production Call Capture       | P1       | Draft   |
| FR-015 | Drift Detection               | P1       | Draft   |
| FR-016 | Rollback Control              | P1       | Draft   |
| FR-017 | Control Dashboard             | P1       | Draft   |

---

## REQUIREMENTS

### FR-001 — Agent Registration

**Description:**
The system must allow users to register an AI agent with a name, description, and associated prompt.

**Acceptance Criteria:**
- User can create an agent with a name and description
- Agent is assigned a unique UUID on creation
- Agent status defaults to `draft` on creation
- Agent can be updated, deactivated, or deleted
- Agent registration does not require a prompt at creation time

**Dependencies:** None
**Priority:** P0

---

### FR-002 — Prompt Module Management

**Description:**
The system must allow prompts to be structured as independent, tagged modules that compose into a full prompt.

**Acceptance Criteria:**
- Prompt is composed of one or more modules
- Supported module tags: SYSTEM, CONTEXT, INSTRUCTIONS, FORMAT, EXAMPLES, GUARDRAILS
- Each module has: tag, name, content, order index, token count
- Modules can be added, edited, reordered, and deleted independently
- Raw unstructured prompt can be submitted and auto-split into modules by the system
- Full assembled prompt is computed from ordered modules on demand

**Dependencies:** FR-001
**Priority:** P0

---

### FR-003 — Test Suite Definition

**Description:**
The system must allow users to define structured test suites that specify what correct agent behaviour looks like.

**Acceptance Criteria:**
- User can create a named test suite linked to an agent
- Test suite contains one or more test cases
- Each test case contains:
  - input: the text or data sent to the agent
  - criteria: list of plain English behaviour expectations
  - validator type per criterion: schema / regex / rule / similarity / llm_judge
  - pass threshold: minimum score to pass (default 0.8)
- Test suite is versioned independently of the prompt
- Test suite can be cloned and modified

**Dependencies:** FR-001
**Priority:** P0

---

### FR-004 — Evaluation Run Execution

**Description:**
The system must execute a test suite against a prompt version and return fully structured results.

**Acceptance Criteria:**
- User triggers eval run by specifying prompt version ID and test suite ID
- Eval engine executes all configured validation layers in order
- Each criterion is scored independently (0.0 to 1.0)
- Overall score is weighted aggregate across all criteria
- Results include: overall score, per-criterion scores, failures, layer breakdown
- Eval run persisted with status: pending / running / passed / failed / error
- Eval run is linked to prompt version for traceability

**Dependencies:** FR-002, FR-003, FR-005
**Priority:** P0

---

### FR-005 — Layer 1: Deterministic Validation

**Description:**
The system must validate agent outputs against deterministic rules without any LLM involvement.

**Acceptance Criteria:**
- JSON schema validation: output matches user-defined JSON schema
- Required field check: specified fields must be present in output
- Forbidden string check: specified strings must not appear in output
- Regex pattern check: output matches or does not match user-defined pattern
- Length bound check: output within user-defined min/max character or token bounds
- Each check returns: pass/fail (boolean), reason (string)
- Layer 1 always runs first before any other layer
- Layer 1 failures are flagged as high severity by default

**Dependencies:** FR-004
**Priority:** P0

---

### FR-006 — Layer 2: Statistical Validation

**Description:**
The system must validate outputs using statistical and similarity methods that do not require LLM calls.

**Acceptance Criteria:**
- Semantic similarity: cosine similarity between output embedding and expected output embedding using pgvector
- Consistency score: same input run N times, variance of outputs measured and scored
- Pattern matching: output matches expected structural patterns defined by user
- All scores returned as 0.0 to 1.0 with short explanation
- Layer 2 runs only after Layer 1 passes or is not configured

**Dependencies:** FR-004, FR-005
**Priority:** P1

---

### FR-007 — Layer 2.5: Reasoning Signal Validation

**Description:**
The system must compute reasoning signals to detect weak, inconsistent, or unreliable agent reasoning without validating reasoning directly.

**Acceptance Criteria:**
- Stability score: input is rephrased N times, output consistency is measured
- Invariant checks: user-defined logical invariants that must always hold regardless of input variation
- LLM critique: LLM is asked to identify logical violations only, not generate reasoning chain
- Reasoning score formula: (0.5 * stability) + (0.3 * constraint) + (0.2 * critique)
- Reasoning score is computed and logged but is NOT a hard gate blocker in V1
- Reasoning signals are surfaced in results as informational, not pass/fail

**Dependencies:** FR-004, FR-006
**Priority:** P1

---

### FR-008 — Layer 3: LLM Judge

**Description:**
The system must support LLM-as-judge evaluation as a final validation layer for intent and semantic correctness.

**Acceptance Criteria:**
- LLM judge fires only when layers 1, 2, and 2.5 do not fully cover all defined criteria
- Judge receives: input, output, and list of unvalidated criteria
- Judge returns structured JSON: overall score, per-criterion score, pass/fail, reason
- Model used for judging is configurable per test suite (Claude or OpenAI)
- Token usage and cost tracked per judge call
- LLM judge result is never the sole determinant of pass/fail when deterministic checks exist

**Dependencies:** FR-004, FR-005, FR-006
**Priority:** P1

---

### FR-009 — Failure Attribution Engine

**Description:**
The system must attribute eval failures to specific prompt modules using perturbation analysis.

**Acceptance Criteria:**
- For each failing eval run, system runs module-level perturbation analysis
- Perturbation process: remove or neutralise one module, re-run eval, record score delta
- Module with highest negative score delta is flagged as primary failure contributor
- Attribution result includes: module tag, module name, contribution score, confidence
- LLM attribution assist: optional semantic hint identifying likely failure modules
- Attribution is persisted as a FailureTrace linked to the eval run
- Attribution runs asynchronously and does not block eval result delivery

**Dependencies:** FR-004, FR-002
**Priority:** P0

---

### FR-010 — Auto Fix Generation

**Description:**
The system must automatically generate an improved prompt variant by rewriting only the failing module.

**Acceptance Criteria:**
- Fix generation is triggered after attribution identifies the failing module
- Fix prompt receives: original module content, failing criteria, failure description
- Fix engine rewrites only the identified failing module, preserving all other modules
- Fixed variant is saved as a new prompt version with parent version reference
- Re-eval is automatically triggered against the same test suite on the fixed variant
- Score comparison between original and fixed variant is returned
- Fix is accepted only if score improves by a configurable threshold (default: +0.1)
- Fix generation and re-eval results are presented side by side

**Dependencies:** FR-009, FR-004
**Priority:** P0

---

### FR-011 — Prompt Version Management

**Description:**
The system must maintain a full version history of every prompt and its modules.

**Acceptance Criteria:**
- Every save creates a new immutable version with an incremented version number
- Version includes: all module contents, model config, created timestamp, created by
- Versions can be compared side by side (diff view)
- Any prior version can be restored as a new version
- Version status: draft / active / archived
- Only one version can be active at a time per agent
- Version history is never deleted

**Dependencies:** FR-002
**Priority:** P0

---

### FR-012 — Deployment Gate

**Description:**
The system must enforce a quality gate that prevents a prompt version from being marked active unless it passes a defined eval suite.

**Acceptance Criteria:**
- User configures a gate by linking a test suite to an agent as the deployment gate
- Prompt version cannot be set to active status without a passing eval run against the gate suite
- Gate pass threshold is configurable (default: 0.8 overall score)
- Attempting to activate a version without a passing eval returns a clear error
- Gate status is visible on the version history view
- Gate can be bypassed with explicit override flag (logged with reason)

**Dependencies:** FR-004, FR-011
**Priority:** P0

---

### FR-013 — Python SDK

**Description:**
The system must provide a lightweight Python SDK that intercepts LLM calls and automatically logs them to AgentForge.

**Acceptance Criteria:**
- SDK installable via pip: pip install agentforge
- Initialised with one line: agentforge.init(api_key="...")
- agentforge.complete() wraps Claude and OpenAI calls with identical interface
- SDK auto-captures: prompt version ID, input, output, latency ms, cost USD, timestamp
- Capture is async and does not add latency to the LLM call
- SDK works with existing code requiring only import and one line change
- SDK fails silently if AgentForge backend is unreachable (never breaks production)

**Dependencies:** FR-014
**Priority:** P1

---

### FR-014 — Production Call Capture

**Description:**
The system must ingest and persist every LLM call event from the SDK as an immutable append-only log.

**Acceptance Criteria:**
- Every SDK call creates a LLMCallEvent record
- Record contains: agent ID, prompt version ID, session ID, input, output, latency, cost, timestamp
- Records are append-only and never modified after creation
- Async eval is triggered on each ingested call against the active gate suite
- Eval score is attached to the call event on completion
- Call events are queryable by agent, version, date range, score range

**Dependencies:** FR-013
**Priority:** P1

---

### FR-015 — Drift Detection

**Description:**
The system must detect when agent output quality degrades over time in production.

**Acceptance Criteria:**
- Drift is computed by comparing recent eval scores against historical baseline per agent
- Drift detection prompt receives: historical scores array, recent scores array
- Drift result: drift detected (boolean), confidence (0.0 to 1.0), reason (string)
- Alert is triggered when drift is detected above configured confidence threshold (default: 0.8)
- Alerts are delivered via: dashboard notification and webhook (configurable)
- Drift is computed on a scheduled basis (default: every 24 hours)

**Dependencies:** FR-014
**Priority:** P1

---

### FR-016 — Rollback Control

**Description:**
The system must allow instant rollback of the active prompt version to any prior passing version.

**Acceptance Criteria:**
- User can view all prior versions with their eval scores and pass/fail status
- User can trigger rollback to any prior version with a passing eval record
- Rollback sets the selected version as active and archives the current active version
- Rollback is logged with: triggered by, timestamp, reason (optional), from version, to version
- Rollback takes effect immediately for all subsequent SDK calls
- Rollback cannot target a version with no passing eval record unless override flag is used

**Dependencies:** FR-011, FR-012
**Priority:** P1

---

### FR-017 — Control Dashboard

**Description:**
The system must provide a dashboard showing production health per agent and per prompt version.

**Acceptance Criteria:**
- Failure rate per prompt version over configurable time window
- Overall quality score trend over time (line chart)
- Top failing input patterns (clustered by similarity)
- Drift status per agent (healthy / warning / critical)
- Cost and latency trends per prompt version
- One-click rollback button per agent
- All data refreshes on configurable interval (default: 5 minutes)

**Dependencies:** FR-014, FR-015, FR-016
**Priority:** P1
