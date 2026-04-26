# af-criteria-template-spec-v1
# Project: AgentForge
# Document Type: Criteria Template Library Specification
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

This document defines the criteria template library for AgentForge.
Templates lower onboarding friction — users get working eval criteria immediately
without having to write validator configs from scratch.

Without this layer, onboarding friction kills adoption before the engine gets tested.

---

## TEMPLATE STRUCTURE

Each template has:

```json
{
  "id": "string — unique template identifier",
  "name": "string — human readable name",
  "description": "string — what this template checks",
  "validator_type": "schema|field|string|regex|length|similarity|invariant|llm_judge|trajectory|tool_precision|cost_gate|latency_gate",
  "default_config": "object — pre-filled validator config, user can override",
  "tags": ["array of tags for filtering"],
  "domain": "general|qa|rag|support|code|extraction|summarisation",
  "layer": "1|2|25|3",
  "severity": "low|medium|high — default severity if this check fails"
}
```

---

## TEMPLATE REGISTRY — LAYER 1 (DETERMINISTIC)

### General Purpose Templates

**TPL-001 — Valid JSON Output**
```json
{
  "id": "TPL-001",
  "name": "Valid JSON Output",
  "description": "Output must be parseable as valid JSON",
  "validator_type": "schema",
  "default_config": { "schema": { "type": "object" } },
  "tags": ["json", "format", "general"],
  "domain": "general",
  "layer": 1,
  "severity": "high"
}
```

**TPL-002 — No Hallucinated URLs**
```json
{
  "id": "TPL-002",
  "name": "No Hallucinated URLs",
  "description": "Output must not contain fabricated URLs",
  "validator_type": "regex",
  "default_config": {
    "pattern": "https?://(?!example\\.com|placeholder)[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}",
    "mode": "must_not_match"
  },
  "tags": ["hallucination", "urls", "general"],
  "domain": "general",
  "layer": 1,
  "severity": "high"
}
```

**TPL-003 — No Placeholder Text**
```json
{
  "id": "TPL-003",
  "name": "No Placeholder Text",
  "description": "Output must not contain placeholder or TODO markers",
  "validator_type": "string",
  "default_config": {
    "forbidden_strings": ["TODO", "FIXME", "PLACEHOLDER", "INSERT HERE", "TBD", "XXX"],
    "case_sensitive": false
  },
  "tags": ["placeholder", "completeness", "general"],
  "domain": "general",
  "layer": 1,
  "severity": "medium"
}
```

**TPL-004 — Minimum Output Length**
```json
{
  "id": "TPL-004",
  "name": "Minimum Output Length",
  "description": "Output must meet minimum length requirements",
  "validator_type": "length",
  "default_config": { "min_chars": 50 },
  "tags": ["completeness", "length", "general"],
  "domain": "general",
  "layer": 1,
  "severity": "medium"
}
```

**TPL-005 — Maximum Output Length**
```json
{
  "id": "TPL-005",
  "name": "Maximum Output Length",
  "description": "Output must not exceed maximum length",
  "validator_type": "length",
  "default_config": { "max_tokens": 2000 },
  "tags": ["conciseness", "length", "general"],
  "domain": "general",
  "layer": 1,
  "severity": "low"
}
```

**TPL-006 — Required Fields Present**
```json
{
  "id": "TPL-006",
  "name": "Required Fields Present",
  "description": "Output JSON must contain all required fields",
  "validator_type": "field",
  "default_config": { "required_fields": [] },
  "tags": ["structure", "fields", "general"],
  "domain": "general",
  "layer": 1,
  "severity": "high"
}
```

**TPL-007 — Cost Gate**
```json
{
  "id": "TPL-007",
  "name": "Cost Gate",
  "description": "Agent call must not exceed maximum cost threshold",
  "validator_type": "cost_gate",
  "default_config": { "max_cost_usd": 0.05 },
  "tags": ["cost", "envelope", "production"],
  "domain": "general",
  "layer": 1,
  "severity": "high"
}
```

**TPL-008 — Latency Gate**
```json
{
  "id": "TPL-008",
  "name": "Latency Gate",
  "description": "Agent call must not exceed maximum latency threshold",
  "validator_type": "latency_gate",
  "default_config": { "max_latency_ms": 5000 },
  "tags": ["latency", "envelope", "production"],
  "domain": "general",
  "layer": 1,
  "severity": "medium"
}
```

---

### QA Automation Templates

**TPL-009 — Test Case Has Setup and Teardown**
```json
{
  "id": "TPL-009",
  "name": "Test Case Has Setup and Teardown",
  "description": "Generated test case must include both setup and teardown sections",
  "validator_type": "string",
  "default_config": {
    "required_strings": ["[Setup]", "[Teardown]"]
  },
  "tags": ["qa", "robot-framework", "structure"],
  "domain": "qa",
  "layer": 1,
  "severity": "high"
}
```

**TPL-010 — No Hardcoded Credentials**
```json
{
  "id": "TPL-010",
  "name": "No Hardcoded Credentials",
  "description": "Test case must not contain hardcoded passwords or API keys",
  "validator_type": "regex",
  "default_config": {
    "pattern": "(password|api_key|secret|token)\\s*=\\s*['\"][^'\"]{4,}['\"]",
    "mode": "must_not_match"
  },
  "tags": ["qa", "security", "credentials"],
  "domain": "qa",
  "layer": 1,
  "severity": "high"
}
```

**TPL-011 — Test Has Documentation Tag**
```json
{
  "id": "TPL-011",
  "name": "Test Has Documentation Tag",
  "description": "Every generated test must include a [Documentation] tag",
  "validator_type": "string",
  "default_config": {
    "forbidden_strings": [],
    "required_strings": ["[Documentation]"]
  },
  "tags": ["qa", "robot-framework", "documentation"],
  "domain": "qa",
  "layer": 1,
  "severity": "medium"
}
```

**TPL-012 — Valid Robot Framework Syntax**
```json
{
  "id": "TPL-012",
  "name": "Valid Robot Framework Syntax",
  "description": "Output must follow Robot Framework file structure",
  "validator_type": "regex",
  "default_config": {
    "pattern": "\\*\\*\\*\\s*(Test Cases|Keywords|Settings|Variables)\\s*\\*\\*\\*",
    "mode": "must_match"
  },
  "tags": ["qa", "robot-framework", "syntax"],
  "domain": "qa",
  "layer": 1,
  "severity": "high"
}
```

---

### RAG / Knowledge Base Templates

**TPL-013 — Answer Cites Source**
```json
{
  "id": "TPL-013",
  "name": "Answer Cites Source",
  "description": "RAG output must reference or cite the source document",
  "validator_type": "field",
  "default_config": { "required_fields": ["source", "citations"] },
  "tags": ["rag", "citation", "faithfulness"],
  "domain": "rag",
  "layer": 1,
  "severity": "high"
}
```

**TPL-014 — No Out-of-Context Claims**
```json
{
  "id": "TPL-014",
  "name": "No Out-of-Context Claims",
  "description": "RAG output must not assert facts not present in retrieved context",
  "validator_type": "llm_judge",
  "default_config": {
    "criteria": "The output must only contain information present in the provided context. Any claim not directly supported by the context is a hallucination.",
    "faithfulness_mode": true
  },
  "tags": ["rag", "hallucination", "faithfulness"],
  "domain": "rag",
  "layer": 3,
  "severity": "high"
}
```

**TPL-015 — Answer Addresses Question**
```json
{
  "id": "TPL-015",
  "name": "Answer Addresses Question",
  "description": "Output must directly address the user question asked",
  "validator_type": "llm_judge",
  "default_config": {
    "criteria": "The output must directly and completely address the specific question asked. A response that is related but does not answer the question fails.",
    "faithfulness_mode": false
  },
  "tags": ["rag", "relevance", "general"],
  "domain": "rag",
  "layer": 3,
  "severity": "high"
}
```

---

## DOMAIN STARTER PACKS

### PACK-001 — QA Automation Agent

**Description:** Pre-built test suite for agents that generate automated test cases.

**Test Cases:**

**TC-1: Basic test generation**
- Input: "Generate a login test case for a web application"
- Criteria: TPL-009 (setup/teardown), TPL-010 (no credentials), TPL-011 (documentation), TPL-012 (RF syntax), TPL-003 (no placeholders)
- Pass threshold: 0.9

**TC-2: Edge case handling**
- Input: "Generate test cases for an invalid login with special characters in username"
- Criteria: TPL-009, TPL-012, TPL-004 (min length), TPL-003
- Pass threshold: 0.85

**TC-3: Test data separation**
- Input: "Generate a data-driven test for multiple user roles"
- Criteria: TPL-010 (no credentials), TPL-012, TPL-001 (valid structure)
- Pass threshold: 0.85

---

### PACK-002 — RAG / Knowledge Base Agent

**Description:** Pre-built test suite for agents that answer questions from documents.

**Test Cases:**

**TC-1: Factual question answering**
- Input: "What is the return policy?" (with context document)
- Criteria: TPL-013 (cites source), TPL-014 (no out-of-context), TPL-015 (addresses question), TPL-004 (min length)
- Pass threshold: 0.9

**TC-2: Out-of-scope question handling**
- Input: "What is the weather today?" (with unrelated context)
- Criteria: TPL-014 (no hallucination), custom: "output must acknowledge it cannot answer from provided context"
- Pass threshold: 0.85

**TC-3: Multi-document synthesis**
- Input: "Summarise the key points from the provided documents"
- Criteria: TPL-013, TPL-014, TPL-015, TPL-005 (max length)
- Pass threshold: 0.85

---

### PACK-003 — General Purpose Agent

**Description:** Baseline test suite suitable for any LLM agent.

**Test Cases:**

**TC-1: Task completion**
- Input: Configurable by user
- Criteria: TPL-003 (no placeholders), TPL-004 (min length), TPL-007 (cost gate), TPL-008 (latency gate)
- Pass threshold: 0.8

**TC-2: Format compliance**
- Input: Configurable by user
- Criteria: TPL-001 (valid JSON if applicable), TPL-006 (required fields if applicable), TPL-005 (max length)
- Pass threshold: 0.8

**TC-3: Robustness**
- Input: Edge case or adversarial input defined by user
- Criteria: TPL-003, TPL-004, TPL-007, TPL-008
- Pass threshold: 0.75

---

## TEMPLATE API ENDPOINTS

```
GET  /criteria-templates                    → list all templates (filter by domain, layer, tags)
GET  /criteria-templates/{id}               → get single template
POST /criteria-templates/apply              → apply template to a test case criterion
GET  /domain-packs                          → list all domain starter packs
POST /domain-packs/{pack_id}/apply          → apply starter pack to agent (creates test suite)
POST /criteria-templates/custom             → save a user-defined custom template
GET  /criteria-templates/custom             → list user's custom templates
```

---

## IMPLEMENTATION NOTES

- Templates are seeded into the database on first startup via migration
- Template IDs are stable and never change once published
- Template default_config is a starting point — user always overrides per criterion
- Custom templates are scoped per user/organisation
- Template library is the primary onboarding mechanism — every new agent creation surfaces relevant templates
