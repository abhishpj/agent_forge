# af-nfr-registry-v1
# Project: AgentForge
# Document Type: Non-Functional Requirements Registry
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

This registry defines all non-functional requirements for AgentForge.
These requirements govern quality attributes: performance, reliability, security, scalability, and maintainability.

---

## REQUIREMENT INDEX

| ID      | Category        | Name                          | Priority |
|---------|-----------------|-------------------------------|----------|
| NFR-001 | Performance     | Eval Run Latency              | P0       |
| NFR-002 | Performance     | Layer 1 Validation Speed      | P0       |
| NFR-003 | Performance     | SDK Overhead                  | P0       |
| NFR-004 | Reliability     | SDK Fault Tolerance           | P0       |
| NFR-005 | Reliability     | Eval Engine Idempotency       | P0       |
| NFR-006 | Reliability     | Data Durability               | P0       |
| NFR-007 | Scalability     | Concurrent Eval Runs          | P1       |
| NFR-008 | Scalability     | Call Event Ingestion Volume   | P1       |
| NFR-009 | Security        | API Authentication            | P0       |
| NFR-010 | Security        | Data Isolation                | P0       |
| NFR-011 | Maintainability | Code Style                    | P0       |
| NFR-012 | Maintainability | Test Coverage                 | P1       |
| NFR-013 | Observability   | System Logging                | P0       |
| NFR-014 | Observability   | Error Tracing                 | P0       |

---

## REQUIREMENTS

### NFR-001 — Eval Run Latency

**Category:** Performance
**Description:** A complete eval run must complete within acceptable time bounds.

**Requirement:**
- Layer 1 only eval: must complete within 2 seconds
- Layer 1 + Layer 2 eval: must complete within 10 seconds
- Full eval (all layers): must complete within 30 seconds
- Attribution analysis: must complete within 60 seconds (async)
- Fix generation + re-eval: must complete within 90 seconds (async)

**Priority:** P0

---

### NFR-002 — Layer 1 Validation Speed

**Category:** Performance
**Description:** Deterministic validators must be fast enough to run synchronously without timeout risk.

**Requirement:**
- Each individual Layer 1 check must complete in under 100ms
- A test suite with 20 Layer 1 checks must complete in under 500ms
- No external calls permitted in Layer 1

**Priority:** P0

---

### NFR-003 — SDK Overhead

**Category:** Performance
**Description:** The SDK must not meaningfully increase latency of instrumented LLM calls.

**Requirement:**
- SDK capture overhead must not exceed 5ms per call
- All capture operations are async and non-blocking
- SDK must not hold or delay the LLM response

**Priority:** P0

---

### NFR-004 — SDK Fault Tolerance

**Category:** Reliability
**Description:** The SDK must never cause production outages or errors in the host application.

**Requirement:**
- SDK fails silently if AgentForge backend is unreachable
- SDK never throws unhandled exceptions into host application
- SDK uses timeout of 2 seconds on all backend calls; drops silently on timeout
- SDK degraded mode: if capture fails, LLM call still succeeds and returns normally

**Priority:** P0

---

### NFR-005 — Eval Engine Idempotency

**Category:** Reliability
**Description:** Running the same eval run twice with the same inputs must produce consistent results.

**Requirement:**
- Layer 1 results must be 100% deterministic (identical inputs = identical results)
- Layer 2 results must be consistent within ±0.05 score variance
- LLM judge results documented as probabilistic; variance acceptable
- Eval run ID is unique; re-running creates a new run, never overwrites

**Priority:** P0

---

### NFR-006 — Data Durability

**Category:** Reliability
**Description:** All persisted data must be durable and recoverable.

**Requirement:**
- All eval runs, call events, and prompt versions are persisted to PostgreSQL
- LLMCallEvent records are append-only and never modified after creation
- Prompt version history is never deleted
- Database backups on daily schedule minimum

**Priority:** P0

---

### NFR-007 — Concurrent Eval Runs

**Category:** Scalability
**Description:** The system must support multiple concurrent eval runs without degradation.

**Requirement:**
- Minimum 10 concurrent eval runs supported in MVP
- Eval runs are isolated: one run's failure does not affect another
- Async workers handle attribution and fix generation independently per run

**Priority:** P1

---

### NFR-008 — Call Event Ingestion Volume

**Category:** Scalability
**Description:** The SDK ingest endpoint must handle production-scale call volumes.

**Requirement:**
- Minimum 100 call events per second ingested without data loss
- Ingest endpoint returns 202 Accepted immediately; processing is async
- Call events are queued via Redis before async worker processes them

**Priority:** P1

---

### NFR-009 — API Authentication

**Category:** Security
**Description:** All API endpoints must require authentication.

**Requirement:**
- All endpoints require API key authentication via Authorization header
- SDK uses a dedicated SDK API key scoped to ingest only
- API keys are hashed before storage; never stored in plain text
- Invalid or missing API key returns 401 Unauthorized

**Priority:** P0

---

### NFR-010 — Data Isolation

**Category:** Security
**Description:** Agent and eval data must be isolated per user or organisation.

**Requirement:**
- All queries are scoped to the authenticated user or organisation
- No cross-user data access is possible through any API endpoint
- User cannot access, modify, or delete another user's agents, prompts, or eval runs

**Priority:** P0

---

### NFR-011 — Code Style

**Category:** Maintainability
**Description:** All code must follow defined style standards for readability and consistency.

**Requirement:**
- Maximum readability at all times
- No one-liners; no nested ternaries
- One responsibility per function
- Full descriptive variable names; no abbreviations
- Functions must not exceed 20 lines
- All functions have docstrings describing purpose, parameters, and return value
- No magic numbers; all constants named and defined at module level

**Priority:** P0

---

### NFR-012 — Test Coverage

**Category:** Maintainability
**Description:** All core engine components must have automated tests.

**Requirement:**
- Layer 1 validators: 100% unit test coverage
- Layer 2 validators: minimum 80% unit test coverage
- Eval engine orchestrator: integration tests for all happy paths and key failure paths
- Attribution engine: unit tests for perturbation logic
- Fix engine: unit tests for prompt rewrite logic
- API endpoints: integration tests for all P0 endpoints

**Priority:** P1

---

### NFR-013 — System Logging

**Category:** Observability
**Description:** All system operations must produce structured logs.

**Requirement:**
- All logs are structured JSON
- Every log entry includes: timestamp, level, component, message, correlation ID
- Eval run logs include: run ID, agent ID, prompt version, layer, score, duration
- SDK ingest logs include: event ID, agent ID, version ID, latency
- Log levels: DEBUG, INFO, WARNING, ERROR — configurable per environment

**Priority:** P0

---

### NFR-014 — Error Tracing

**Category:** Observability
**Description:** All errors must be traceable to their source with full context.

**Requirement:**
- Every error includes: correlation ID, component, operation, input context, stack trace
- Eval engine errors are caught per layer and do not halt subsequent layers
- Attribution engine errors are caught and do not block eval result delivery
- All unhandled exceptions are logged before propagation
- Error responses to API clients include correlation ID for support tracing

**Priority:** P0
