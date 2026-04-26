# af-component-spec-v1
# Project: AgentForge
# Document Type: Component Specification
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

This document defines all components of the AgentForge system, their responsibilities,
interfaces, and boundaries. Every module built must map to a component defined here.

---

## SYSTEM OVERVIEW

```
agentforge/
├── AGENTS.md                    # ADF permanent rules file for coding agents
├── docs/
│   ├── s1-requirements/         # Gates, FR, NFR, AC registries
│   └── specs/                   # Component, data contract, prompt, impl specs
├── backend/
│   ├── main.py                  # FastAPI app entry point
│   ├── core/
│   │   ├── config.py            # Environment config and constants
│   │   ├── logger.py            # Structured JSON logger
│   │   ├── database.py          # SQLAlchemy engine and session factory
│   │   └── exceptions.py        # Custom exception classes
│   ├── models/
│   │   ├── agent.py             # Agent ORM model
│   │   ├── prompt.py            # PromptVersion and PromptModule ORM models
│   │   ├── test_suite.py        # TestSuite and TestCase ORM models
│   │   ├── eval_run.py          # EvalRun and EvalResult ORM models
│   │   ├── failure_trace.py     # FailureTrace ORM model
│   │   ├── fix_variant.py       # FixVariant ORM model
│   │   └── call_event.py        # LLMCallEvent ORM model
│   ├── schemas/
│   │   ├── agent.py             # Pydantic request/response schemas for agents
│   │   ├── prompt.py            # Pydantic schemas for prompts and modules
│   │   ├── test_suite.py        # Pydantic schemas for test suites
│   │   ├── eval_run.py          # Pydantic schemas for eval runs and results
│   │   └── sdk.py               # Pydantic schemas for SDK ingest
│   ├── api/
│   │   ├── agents.py            # Agent CRUD endpoints
│   │   ├── prompts.py           # Prompt and module endpoints
│   │   ├── test_suites.py       # Test suite endpoints
│   │   ├── evals.py             # Eval run endpoints
│   │   ├── fixes.py             # Fix generation endpoints
│   │   ├── dashboard.py         # Control dashboard endpoints
│   │   └── sdk_ingest.py        # SDK call event ingest endpoint
│   ├── engine/
│   │   ├── orchestrator.py      # Eval run orchestrator — coordinates all layers
│   │   ├── layer1/
│   │   │   ├── runner.py        # Layer 1 runner — executes all deterministic checks
│   │   │   ├── schema_validator.py     # JSON schema validation
│   │   │   ├── field_validator.py      # Required field checks
│   │   │   ├── string_validator.py     # Forbidden string checks
│   │   │   ├── regex_validator.py      # Regex pattern checks
│   │   │   └── length_validator.py     # Length bound checks
│   │   ├── layer2/
│   │   │   ├── runner.py        # Layer 2 runner — statistical checks
│   │   │   ├── similarity_validator.py # Cosine similarity via pgvector
│   │   │   └── consistency_validator.py # Output consistency across runs
│   │   ├── layer25/
│   │   │   ├── runner.py        # Layer 2.5 runner — reasoning signals
│   │   │   ├── stability_checker.py    # Input rephrasing + output variance
│   │   │   ├── invariant_checker.py    # Logical invariant validation
│   │   │   └── critique_checker.py     # LLM logical violation detection
│   │   ├── layer3/
│   │   │   ├── runner.py        # Layer 3 runner — LLM judge
│   │   │   └── llm_judge.py     # LLM-as-judge evaluation
│   │   ├── attribution/
│   │   │   ├── engine.py        # Perturbation-based attribution orchestrator
│   │   │   └── scorer.py        # Delta scoring and module blame assignment
│   │   └── fix/
│   │       ├── generator.py     # Fix variant generation
│   │       └── verifier.py      # Re-eval and score comparison
│   ├── llm/
│   │   ├── client.py            # Unified LLM client (Claude + OpenAI abstraction)
│   │   └── prompts/
│   │       ├── eval_judge.py    # LLM judge system prompt
│   │       ├── failure_classify.py  # Failure classification prompt
│   │       ├── fix_generate.py  # Fix generation prompt
│   │       ├── attribution.py   # Attribution assist prompt
│   │       └── drift_detect.py  # Drift detection prompt
│   └── migrations/
│       └── versions/            # Alembic migration files
├── sdk/
│   ├── agentforge/
│   │   ├── __init__.py          # SDK public API: init(), complete()
│   │   ├── client.py            # LLM call wrapper
│   │   ├── capture.py           # Async call event capture
│   │   └── logger.py            # SDK internal logger
│   └── setup.py                 # pip package setup
└── frontend/
    ├── src/
    │   ├── pages/
    │   │   ├── Agents.jsx        # Agent list and creation
    │   │   ├── PromptEditor.jsx  # Prompt module editor
    │   │   ├── TestSuites.jsx    # Test suite builder
    │   │   ├── EvalRuns.jsx      # Eval run results
    │   │   └── Dashboard.jsx     # Control dashboard
    │   ├── components/
    │   │   ├── ModuleCard.jsx    # Individual prompt module card
    │   │   ├── CriteriaBuilder.jsx  # Test criteria builder
    │   │   ├── ScoreBreakdown.jsx   # Layer-by-layer score display
    │   │   ├── AttributionView.jsx  # Module blame visualisation
    │   │   └── FixComparison.jsx    # Side-by-side fix comparison
    │   └── api/
    │       └── client.js         # Frontend API client
    └── public/
```

---

## COMPONENT RESPONSIBILITIES

### core/config.py
- Loads all environment variables on startup
- Exposes typed config object used across all components
- Never accessed via os.environ directly outside this file

### core/logger.py
- Structured JSON logger used by all components
- Every log entry includes: timestamp, level, component, message, correlation_id
- Log level configurable via environment variable

### core/database.py
- SQLAlchemy async engine and session factory
- Provides get_db() dependency for FastAPI endpoints
- Connection pool configuration

### engine/orchestrator.py
- Receives eval run request
- Coordinates execution of layers 1, 2, 2.5, 3 in order
- Aggregates results into final EvalResult
- Triggers attribution and fix generation as background tasks
- Never contains validation logic itself

### engine/layer1/runner.py
- Receives output and list of Layer 1 criteria
- Instantiates and runs the correct validator per criterion type
- Returns list of LayerResult objects
- Never calls external services

### engine/attribution/engine.py
- Receives failing eval run ID
- Fetches prompt modules and test suite
- Runs perturbation loop: remove one module, re-run eval, record delta
- Calls scorer.py to assign blame
- Persists FailureTrace records

### engine/fix/generator.py
- Receives attribution result identifying failing module
- Calls LLM fix prompt with module content and failing criteria
- Returns updated module content only
- Never modifies other modules

### llm/client.py
- Single interface for all LLM calls across the system
- Supports Claude and OpenAI via configuration
- Tracks token usage and cost per call
- Raises typed exceptions on API errors

### sdk/agentforge/client.py
- Wraps anthropic.messages.create() and openai.chat.completions.create()
- Calls capture.py async after every LLM call
- Never delays or blocks the LLM response
- Fails silently if capture fails

---

## COMPONENT BOUNDARIES

These boundaries must never be crossed:

- Engine components never import from API components
- API components never contain business logic
- Layer runners never call other layer runners
- SDK never imports from backend directly
- LLM client is the only component that calls external LLM APIs
- Models never contain business logic
