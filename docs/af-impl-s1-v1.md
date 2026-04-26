# af-impl-s1-v1
# Project: AgentForge
# Document Type: Sprint 1 Implementation Specification
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

This document defines the exact implementation scope for AgentForge Sprint 1.
It maps requirements to files to tasks to exit criteria.
The coding agent must implement everything in this document — nothing more, nothing less.

---

## SPRINT 1 SCOPE

Sprint 1 delivers the Eval Core — the sellable MVP.

**In scope:**
- Project folder structure
- Database models and migrations
- FastAPI app skeleton
- Agent CRUD API
- Prompt version and module API
- Test suite and criteria API
- Eval run execution (Layer 1 only in Sprint 1)
- All Layer 1 validators
- Eval result persistence
- Gate enforcement

**Out of scope for Sprint 1:**
- Layer 2, 2.5, 3 (Sprint 2)
- Attribution engine (Sprint 2)
- Fix generation (Sprint 2)
- SDK (Sprint 3)
- Dashboard (Sprint 4)
- Frontend (Sprint 2+)

---

## TASK LIST

### T-001 — Project Structure
**Maps to:** af-component-spec-v1
**Description:** Create the full folder and file skeleton for the backend

**Files to create:**
```
backend/
├── main.py
├── core/
│   ├── __init__.py
│   ├── config.py
│   ├── logger.py
│   ├── database.py
│   └── exceptions.py
├── models/
│   ├── __init__.py
│   ├── agent.py
│   ├── prompt.py
│   ├── test_suite.py
│   ├── eval_run.py
│   └── failure_trace.py
├── schemas/
│   ├── __init__.py
│   ├── agent.py
│   ├── prompt.py
│   ├── test_suite.py
│   └── eval_run.py
├── api/
│   ├── __init__.py
│   ├── agents.py
│   ├── prompts.py
│   ├── test_suites.py
│   └── evals.py
├── engine/
│   ├── __init__.py
│   ├── orchestrator.py
│   └── layer1/
│       ├── __init__.py
│       ├── runner.py
│       ├── schema_validator.py
│       ├── field_validator.py
│       ├── string_validator.py
│       ├── regex_validator.py
│       └── length_validator.py
├── llm/
│   ├── __init__.py
│   └── client.py
├── migrations/
│   └── versions/
├── requirements.txt
├── alembic.ini
└── .env.example
```

**Exit Criteria:** All files created; all imports resolve; app starts without error.

---

### T-002 — Core Configuration
**Maps to:** NFR-011, NFR-013
**File:** backend/core/config.py

**Must implement:**
- Load all environment variables using pydantic-settings
- Expose typed Settings object
- Variables required: DATABASE_URL, REDIS_URL, ANTHROPIC_API_KEY, OPENAI_API_KEY, LOG_LEVEL, API_KEY_HASH_SECRET

**Exit Criteria:** Settings loads correctly from .env; missing required vars raise clear error on startup.

---

### T-003 — Structured Logger
**Maps to:** NFR-013, NFR-014
**File:** backend/core/logger.py

**Must implement:**
- JSON structured logging using Python logging module
- Every log entry includes: timestamp, level, component, message, correlation_id
- get_logger(component_name) function returns configured logger
- Log level driven by config

**Exit Criteria:** Logger produces valid JSON log lines; correlation_id present on all entries.

---

### T-004 — Database Setup
**Maps to:** af-data-contract-spec-v1
**File:** backend/core/database.py

**Must implement:**
- SQLAlchemy async engine configured from DATABASE_URL
- Async session factory
- get_db() FastAPI dependency
- Base declarative model class

**Exit Criteria:** Database connection established; get_db() yields working session.

---

### T-005 — ORM Models
**Maps to:** af-data-contract-spec-v1
**Files:** backend/models/*.py

**Must implement:**
- Agent model
- PromptVersion model
- PromptModule model
- TestSuite model
- TestCase model
- EvalCriteria model
- EvalRun model
- EvalResult model

**Each model must:**
- Match exactly the columns in af-data-contract-spec-v1
- Use UUID primary keys
- Include created_at and updated_at where specified
- Define __tablename__ explicitly

**Exit Criteria:** All models importable; Alembic generates correct migration from models.

---

### T-006 — Database Migration
**Maps to:** af-data-contract-spec-v1
**Directory:** backend/migrations/

**Must implement:**
- Alembic configured to use async SQLAlchemy engine
- Initial migration creating all Sprint 1 tables
- pgvector extension created in migration (for future use)

**Exit Criteria:** alembic upgrade head runs without error; all tables created in database.

---

### T-007 — FastAPI App Entry Point
**Maps to:** af-component-spec-v1
**File:** backend/main.py

**Must implement:**
- FastAPI app instantiation
- CORS middleware configuration
- API key authentication middleware
- Router registration for all Sprint 1 API modules
- Health check endpoint: GET /health → { "status": "ok" }
- Lifespan handler for startup/shutdown

**Exit Criteria:** App starts; GET /health returns 200; unauthenticated request returns 401.

---

### T-008 — Agent API
**Maps to:** FR-001
**File:** backend/api/agents.py

**Endpoints:**
- POST /agents → create agent → 201
- GET /agents → list agents → 200
- GET /agents/{id} → get agent → 200 or 404
- PUT /agents/{id} → update agent → 200 or 404
- DELETE /agents/{id} → delete agent → 204 or 404

**Each endpoint must:**
- Validate request with Pydantic schema
- Return typed Pydantic response schema
- Log operation with component logger
- Handle not-found with 404 and clear message

**Exit Criteria:** All 5 endpoints pass AC-001.

---

### T-009 — Prompt and Module API
**Maps to:** FR-002, FR-011
**File:** backend/api/prompts.py

**Endpoints:**
- POST /prompts → create prompt version with modules → 201
- GET /prompts/{id} → get prompt version → 200 or 404
- GET /prompts/{id}/versions → list all versions for agent → 200
- GET /prompts/{id}/assembled → return full assembled prompt text → 200
- POST /prompts/{id}/modules → add module to version → 201
- PUT /prompts/{id}/modules/{module_id} → update module → 200
- DELETE /prompts/{id}/modules/{module_id} → delete module → 204
- POST /prompts/parse → auto-split raw prompt into modules → 200
- POST /prompts/{id}/activate → activate version (gate enforced) → 200 or 422

**Exit Criteria:** All endpoints pass AC-002, AC-003, AC-011, AC-012.

---

### T-010 — Test Suite API
**Maps to:** FR-003
**File:** backend/api/test_suites.py

**Endpoints:**
- POST /test-suites → create test suite with test cases and criteria → 201
- GET /test-suites/{id} → get test suite with all test cases → 200
- GET /agents/{agent_id}/test-suites → list suites for agent → 200
- PUT /test-suites/{id} → update suite → 200
- DELETE /test-suites/{id} → delete suite → 204

**Exit Criteria:** All endpoints pass AC-004.

---

### T-011 — Layer 1 Validators
**Maps to:** FR-005
**Files:** backend/engine/layer1/*.py

**Must implement:**

**schema_validator.py**
- Validates output against JSON schema using jsonschema library
- Returns: LayerCheckResult(passed, reason)

**field_validator.py**
- Checks all required fields are present in output
- Works on JSON outputs and plain text outputs
- Returns: LayerCheckResult(passed, reason, missing_fields)

**string_validator.py**
- Checks output does not contain forbidden strings
- Case-insensitive matching
- Returns: LayerCheckResult(passed, reason, matched_strings)

**regex_validator.py**
- Checks output matches or does not match a regex pattern
- Mode: must_match or must_not_match
- Returns: LayerCheckResult(passed, reason)

**length_validator.py**
- Checks output character count or token count is within bounds
- Returns: LayerCheckResult(passed, reason, actual_length)

**runner.py**
- Receives list of Layer1Criterion objects
- Instantiates correct validator per criterion type
- Returns list of LayerCheckResult objects
- Never calls external services

**Exit Criteria:** All validators pass AC-006, AC-007, AC-008, AC-013.

---

### T-012 — Eval Run Orchestrator
**Maps to:** FR-004
**File:** backend/engine/orchestrator.py

**Must implement:**
- receive_eval_request(run_id) — loads run, prompt version, test suite from DB
- execute_layer1(prompt_version, test_cases) — runs all Layer 1 checks
- aggregate_results(layer_results) — computes weighted overall score
- persist_results(run_id, results) — saves EvalResult records to DB
- update_run_status(run_id, status, score) — updates EvalRun record

**Sprint 1:** Only Layer 1 is executed. Layers 2, 2.5, 3 are stubbed (return None).

**Exit Criteria:** Eval run executes end-to-end; results persisted; AC-005 passes.

---

### T-013 — Eval Run API
**Maps to:** FR-004
**File:** backend/api/evals.py

**Endpoints:**
- POST /evals/run → trigger eval run → 202 with run_id
- GET /evals/{run_id} → get eval results → 200 or 404
- GET /agents/{agent_id}/evals → list eval runs for agent → 200

**Exit Criteria:** Eval triggered and results retrievable; AC-005 passes.

---

### T-014 — LLM Client Stub
**Maps to:** Component spec llm/client.py
**File:** backend/llm/client.py

**Sprint 1:** Stub only — used for prompt auto-split (EP-006) and nothing else.

**Must implement:**
- LLMClient class with complete(system_prompt, user_prompt, model) method
- Claude support via anthropic SDK
- Returns: LLMResponse(content, input_tokens, output_tokens, cost_usd)
- Raises typed LLMError on API failure

**Exit Criteria:** Client calls Claude API successfully; returns typed response.

---

## REQUIREMENTS.TXT

```
fastapi==0.111.0
uvicorn[standard]==0.29.0
sqlalchemy[asyncio]==2.0.30
asyncpg==0.29.0
alembic==1.13.1
pydantic==2.7.1
pydantic-settings==2.2.1
anthropic==0.25.0
openai==1.30.0
jsonschema==4.22.0
redis==5.0.4
python-multipart==0.0.9
httpx==0.27.0
pytest==8.2.0
pytest-asyncio==0.23.6
```

---

## EXIT CRITERIA FOR SPRINT 1

Sprint 1 is complete when ALL of the following are true:

- [ ] All T-001 through T-014 tasks complete
- [ ] GET /health returns 200
- [ ] All AC-001 through AC-008 acceptance criteria pass
- [ ] AC-012 deployment gate enforcement passes
- [ ] AC-013 Layer 1 performance test passes (20 checks under 500ms)
- [ ] AC-015 code style review passes
- [ ] All functions have docstrings
- [ ] No function exceeds 20 lines
- [ ] Gate 2 approval obtained before any disk write
