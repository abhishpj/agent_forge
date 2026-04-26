# af-data-contract-spec-v1
# Project: AgentForge
# Document Type: Data Contract Specification
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

This document defines all database tables, columns, types, constraints, and indexes for AgentForge.
All ORM models and migrations must exactly match this specification.

---

## DATABASE CONFIGURATION

- Engine: PostgreSQL 16
- Extensions required: pgvector, uuid-ossp
- Default schema: public
- UUID generation: gen_random_uuid()
- Timestamps: TIMESTAMPTZ with DEFAULT NOW()

---

## TABLES

### agents

| Column      | Type        | Constraints                    | Description                    |
|-------------|-------------|--------------------------------|--------------------------------|
| id          | UUID        | PRIMARY KEY, DEFAULT gen_random_uuid() | Unique agent identifier |
| name        | TEXT        | NOT NULL                       | Human readable agent name      |
| description | TEXT        | NULLABLE                       | Agent purpose description      |
| status      | TEXT        | NOT NULL, DEFAULT 'draft'      | draft / active / archived      |
| created_at  | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()        | Creation timestamp             |
| updated_at  | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()        | Last update timestamp          |

**Indexes:**
- PRIMARY KEY on id
- INDEX on status

---

### prompt_versions

| Column         | Type        | Constraints                    | Description                        |
|----------------|-------------|--------------------------------|------------------------------------|
| id             | UUID        | PRIMARY KEY                    | Unique version identifier          |
| agent_id       | UUID        | NOT NULL, FK agents(id)        | Owning agent                       |
| version_number | INTEGER     | NOT NULL                       | Monotonically increasing per agent |
| status         | TEXT        | NOT NULL, DEFAULT 'draft'      | draft / active / archived          |
| model          | TEXT        | NOT NULL                       | Default LLM model for this version |
| created_at     | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()        | Creation timestamp                 |
| created_by     | TEXT        | NULLABLE                       | User who created this version      |
| parent_version_id | UUID     | NULLABLE, FK prompt_versions(id) | Prior version this was derived from |

**Indexes:**
- PRIMARY KEY on id
- INDEX on agent_id
- UNIQUE on (agent_id, version_number)

---

### prompt_modules

| Column           | Type    | Constraints                         | Description                           |
|------------------|---------|-------------------------------------|---------------------------------------|
| id               | UUID    | PRIMARY KEY                         | Unique module identifier              |
| prompt_version_id| UUID    | NOT NULL, FK prompt_versions(id)    | Owning prompt version                 |
| tag              | TEXT    | NOT NULL                            | SYSTEM / CONTEXT / INSTRUCTIONS / FORMAT / EXAMPLES / GUARDRAILS |
| name             | TEXT    | NOT NULL                            | Human readable module name            |
| content          | TEXT    | NOT NULL                            | Module text content                   |
| order_index      | INTEGER | NOT NULL                            | Position in assembled prompt          |
| token_count      | INTEGER | NULLABLE                            | Approximate token count of content    |

**Indexes:**
- PRIMARY KEY on id
- INDEX on prompt_version_id
- INDEX on (prompt_version_id, order_index)

---

### test_suites

| Column      | Type        | Constraints             | Description                     |
|-------------|-------------|-------------------------|---------------------------------|
| id          | UUID        | PRIMARY KEY             | Unique test suite identifier    |
| agent_id    | UUID        | NOT NULL, FK agents(id) | Owning agent                    |
| name        | TEXT        | NOT NULL                | Human readable suite name       |
| description | TEXT        | NULLABLE                | Suite purpose                   |
| pass_threshold | FLOAT    | NOT NULL, DEFAULT 0.8   | Minimum score to pass           |
| is_gate     | BOOLEAN     | NOT NULL, DEFAULT FALSE | Whether this is the deployment gate |
| created_at  | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Creation timestamp              |

**Indexes:**
- PRIMARY KEY on id
- INDEX on agent_id

---

### test_cases

| Column         | Type    | Constraints                     | Description                          |
|----------------|---------|----------------------------------|--------------------------------------|
| id             | UUID    | PRIMARY KEY                      | Unique test case identifier          |
| test_suite_id  | UUID    | NOT NULL, FK test_suites(id)     | Owning test suite                    |
| name           | TEXT    | NOT NULL                         | Human readable test case name        |
| input          | TEXT    | NOT NULL                         | Input sent to the agent              |
| pass_threshold | FLOAT   | NOT NULL, DEFAULT 0.8            | Minimum score for this test case     |
| order_index    | INTEGER | NOT NULL                         | Execution order within suite         |

**Indexes:**
- PRIMARY KEY on id
- INDEX on test_suite_id

---

### eval_criteria

| Column          | Type    | Constraints                  | Description                             |
|-----------------|---------|------------------------------|-----------------------------------------|
| id              | UUID    | PRIMARY KEY                  | Unique criterion identifier             |
| test_case_id    | UUID    | NOT NULL, FK test_cases(id)  | Owning test case                        |
| description     | TEXT    | NOT NULL                     | Plain English criterion statement       |
| validator_type  | TEXT    | NOT NULL                     | schema / regex / field / length / string / similarity / invariant / llm_judge |
| validator_config| JSONB   | NOT NULL                     | Validator-specific configuration        |
| weight          | FLOAT   | NOT NULL, DEFAULT 1.0        | Weight in overall score calculation     |
| layer           | INTEGER | NOT NULL                     | 1 / 2 / 25 / 3                          |

**Indexes:**
- PRIMARY KEY on id
- INDEX on test_case_id

---

### eval_runs

| Column            | Type        | Constraints                       | Description                          |
|-------------------|-------------|-----------------------------------|--------------------------------------|
| id                | UUID        | PRIMARY KEY                       | Unique run identifier                |
| prompt_version_id | UUID        | NOT NULL, FK prompt_versions(id)  | Prompt version evaluated             |
| test_suite_id     | UUID        | NOT NULL, FK test_suites(id)      | Test suite used                      |
| overall_score     | FLOAT       | NULLABLE                          | Final weighted score (0.0 to 1.0)    |
| status            | TEXT        | NOT NULL, DEFAULT 'pending'       | pending / running / passed / failed / error |
| layer1_score      | FLOAT       | NULLABLE                          | Layer 1 aggregate score              |
| layer2_score      | FLOAT       | NULLABLE                          | Layer 2 aggregate score              |
| layer25_score     | FLOAT       | NULLABLE                          | Layer 2.5 reasoning signal score     |
| layer3_score      | FLOAT       | NULLABLE                          | Layer 3 LLM judge score              |
| started_at        | TIMESTAMPTZ | NULLABLE                          | Execution start timestamp            |
| completed_at      | TIMESTAMPTZ | NULLABLE                          | Execution completion timestamp       |
| created_at        | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()           | Run creation timestamp               |

**Indexes:**
- PRIMARY KEY on id
- INDEX on prompt_version_id
- INDEX on test_suite_id
- INDEX on status

---

### eval_results

| Column        | Type    | Constraints                  | Description                          |
|---------------|---------|------------------------------|--------------------------------------|
| id            | UUID    | PRIMARY KEY                  | Unique result identifier             |
| eval_run_id   | UUID    | NOT NULL, FK eval_runs(id)   | Owning eval run                      |
| test_case_id  | UUID    | NOT NULL, FK test_cases(id)  | Test case evaluated                  |
| criteria_id   | UUID    | NOT NULL, FK eval_criteria(id)| Criterion evaluated                 |
| score         | FLOAT   | NOT NULL                     | Score for this criterion (0.0 to 1.0)|
| passed        | BOOLEAN | NOT NULL                     | Whether criterion passed             |
| reason        | TEXT    | NULLABLE                     | Explanation of result                |
| layer         | INTEGER | NOT NULL                     | Which layer produced this result     |

**Indexes:**
- PRIMARY KEY on id
- INDEX on eval_run_id
- INDEX on criteria_id

---

### failure_traces

| Column             | Type    | Constraints                    | Description                              |
|--------------------|---------|--------------------------------|------------------------------------------|
| id                 | UUID    | PRIMARY KEY                    | Unique trace identifier                  |
| eval_run_id        | UUID    | NOT NULL, FK eval_runs(id)     | Failing eval run                         |
| module_id          | UUID    | NOT NULL, FK prompt_modules(id)| Module attributed as failure contributor |
| module_tag         | TEXT    | NOT NULL                       | Tag of attributed module                 |
| contribution_score | FLOAT   | NOT NULL                       | Score delta caused by this module        |
| confidence         | FLOAT   | NOT NULL                       | Attribution confidence (0.0 to 1.0)      |
| failure_types      | TEXT[]  | NOT NULL                       | List of failure type strings             |
| explanation        | TEXT    | NULLABLE                       | Plain English attribution explanation    |
| created_at         | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()    | Trace creation timestamp                 |

**Indexes:**
- PRIMARY KEY on id
- INDEX on eval_run_id

---

### fix_variants

| Column              | Type        | Constraints                       | Description                               |
|---------------------|-------------|-----------------------------------|-------------------------------------------|
| id                  | UUID        | PRIMARY KEY                       | Unique variant identifier                 |
| source_eval_run_id  | UUID        | NOT NULL, FK eval_runs(id)        | Failing eval run that triggered fix       |
| original_version_id | UUID        | NOT NULL, FK prompt_versions(id)  | Original prompt version                   |
| fixed_version_id    | UUID        | NULLABLE, FK prompt_versions(id)  | New prompt version with fix applied       |
| changed_module_id   | UUID        | NOT NULL, FK prompt_modules(id)   | Module that was rewritten                 |
| changes_summary     | TEXT[]      | NOT NULL                          | List of change descriptions               |
| original_score      | FLOAT       | NOT NULL                          | Eval score before fix                     |
| fixed_score         | FLOAT       | NULLABLE                          | Eval score after fix                      |
| score_delta         | FLOAT       | NULLABLE                          | fixed_score minus original_score          |
| accepted            | BOOLEAN     | NULLABLE                          | Whether fix was accepted                  |
| created_at          | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()           | Fix creation timestamp                    |

**Indexes:**
- PRIMARY KEY on id
- INDEX on source_eval_run_id

---

### llm_call_events

| Column            | Type        | Constraints                       | Description                           |
|-------------------|-------------|-----------------------------------|---------------------------------------|
| id                | UUID        | PRIMARY KEY                       | Unique event identifier               |
| agent_id          | UUID        | NOT NULL, FK agents(id)           | Agent that made the call              |
| prompt_version_id | UUID        | NULLABLE, FK prompt_versions(id)  | Prompt version used (if tracked)      |
| session_id        | TEXT        | NULLABLE                          | Session identifier from SDK           |
| input             | TEXT        | NOT NULL                          | Input sent to LLM                     |
| output            | TEXT        | NOT NULL                          | Output received from LLM              |
| model             | TEXT        | NOT NULL                          | LLM model used                        |
| latency_ms        | INTEGER     | NOT NULL                          | LLM call latency in milliseconds      |
| cost_usd          | FLOAT       | NULLABLE                          | Estimated cost in USD                 |
| eval_score        | FLOAT       | NULLABLE                          | Async eval score (attached after eval)|
| ts                | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()           | Event timestamp                       |

**IMPORTANT:** This table is append-only. No UPDATE or DELETE operations are permitted.

**Indexes:**
- PRIMARY KEY on id
- INDEX on agent_id
- INDEX on prompt_version_id
- INDEX on ts
- INDEX on eval_score

---

### prompt_embeddings

| Column            | Type        | Constraints                       | Description                           |
|-------------------|-------------|-----------------------------------|---------------------------------------|
| id                | UUID        | PRIMARY KEY                       | Unique embedding identifier           |
| module_id         | UUID        | NOT NULL, FK prompt_modules(id)   | Module this embedding represents      |
| embedding         | vector(1536)| NOT NULL                          | pgvector embedding (OpenAI 1536-dim)  |
| model             | TEXT        | NOT NULL                          | Embedding model used                  |
| created_at        | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()           | Creation timestamp                    |

**Indexes:**
- PRIMARY KEY on id
- HNSW INDEX on embedding using vector_cosine_ops

---

## ENUMERATIONS

### agent.status
- `draft` — agent created but not configured
- `active` — agent is deployed and operational
- `archived` — agent is decommissioned

### prompt_version.status
- `draft` — version in progress, not evaluated
- `active` — current production version
- `archived` — superseded version

### eval_run.status
- `pending` — run queued, not started
- `running` — evaluation in progress
- `passed` — overall score above threshold
- `failed` — overall score below threshold
- `error` — execution error occurred

### eval_criteria.validator_type
- `schema` — JSON schema validation
- `field` — required field presence check
- `string` — forbidden string check
- `regex` — pattern match check
- `length` — output length bounds check
- `similarity` — semantic similarity check
- `invariant` — logical invariant check
- `llm_judge` — LLM-as-judge evaluation

### eval_criteria.layer
- `1` — deterministic layer
- `2` — statistical layer
- `25` — reasoning signals layer
- `3` — LLM judge layer
