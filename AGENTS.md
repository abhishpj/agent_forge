# AGENTS.md
# Project: AgentForge
# Document Type: Permanent ADF Rules File for Coding Agents
# Version: 2.0
# Status: Active
# Last Updated: 2026-04
# Change: v2.0 — corrected execution sequence, added criteria template layer,
#         refined SDK positioning, split Sprint 2 into 2a/2b/2c

---

## WHAT IS THIS FILE

This is the permanent rules file for all coding agents working on AgentForge.
Read this file completely before writing any code, creating any file, or making any decision.
This file overrides any other instruction if there is a conflict.

---

## WHAT WE ARE BUILDING

AgentForge is a CI/CD + observability + control system for non-deterministic AI behaviour.

It brings the same discipline software teams apply to code — unit tests, CI pipelines,
staging gates, production monitoring, rollback — to AI agent behaviour.

One sentence:
> "We evaluate, attribute, fix, and control AI agent behaviour in production."

---

## CRITICAL EXECUTION RULE — READ FIRST

### The single point of failure for this product is:
> If evaluation + failure attribution is weak, everything collapses.

### The correct execution sequence is NON-NEGOTIABLE:

```
Sprint 1   →  Layer 1 + API + DB + Criteria Templates
              ↓
              VALIDATE on real agent before proceeding
              ↓
Sprint 2a  →  Attribution engine ONLY
              ↓
              MANUALLY VALIDATE attribution accuracy on real failures
              ↓
Sprint 2b  →  Fix engine (ONLY after attribution validated)
              ↓
Sprint 2c  →  Layers 2, 2.5, 3
              ↓
Sprint 3   →  SDK
              ↓
Sprint 4   →  Control dashboard
```

### Why this order matters:
- Fix engine is useless if attribution is wrong
- Attribution must be manually validated before fix is built
- Do NOT build fix and attribution together
- Do NOT start SDK before eval + attribution are proven

---

## ADF GATES — NON-NEGOTIABLE

### Gate 1 — Before writing any code
- Read af-g1-gate-v1.md
- All open questions resolved (Gate 1 is APPROVED)
- All architectural decisions confirmed

### Gate 2 — Before writing any file to disk
- Present all generated code to human for review
- Human must explicitly approve
- Only after approval write files to disk
- Gate 2 is per sprint — repeat for every sprint

### Gate 3 — NEW — Between Sprint 2a and Sprint 2b
- Attribution engine must be manually validated on at least 3 real agent failures
- Attribution must correctly identify the failing module with confidence > 0.7
- Human reviewer must sign off before fix engine is built
- This gate cannot be bypassed

---

## PRODUCT PRINCIPLES

1. Deterministic before probabilistic — always run cheap fast checks first
2. LLM judge is last resort only — never use LLM where a rule check suffices
3. Failures must be attributed — a score without a cause is useless
4. Attribution must be validated before fix is built — non-negotiable sequence
5. Fix must be minimal — rewrite only what is broken, preserve everything else
6. Reasoning signals are informational only — never a hard gate blocker
7. Domain agnostic — never hardcode domain assumptions
8. Production-first thinking — every feature must work at production scale
9. SDK fails silently — never break the host application under any circumstance
10. Criteria templates lower onboarding friction — every domain gets starter packs

---

## CODE STYLE — NON-NEGOTIABLE

1. Maximum readability at all times
2. No function exceeds 20 lines
3. No one-liners for complex logic
4. No nested ternaries
5. One responsibility per function
6. Full descriptive variable names — no abbreviations
7. All functions have docstrings (purpose, parameters, returns)
8. No magic numbers — all constants named at module level
9. No raw SQL — ORM only
10. No os.environ access outside core/config.py

---

## COMPONENT BOUNDARIES — NON-NEGOTIABLE

- Engine components never import from API components
- API components never contain business logic
- Layer runners never call other layer runners directly
- Attribution engine never calls fix engine
- Fix engine never calls attribution engine directly (receives attribution result only)
- SDK never imports from backend
- LLM client is the only component that calls external LLM APIs
- Models never contain business logic

---

## SDK POSITIONING — IMPORTANT

The SDK is NOT "zero config".
The correct positioning is: "Near-zero integration, safe by default"

The one-line change is:
```python
# Before
response = anthropic.messages.create(...)

# After — near-zero integration
response = agentforge.complete(model="claude", prompt_version="v1", input=data)
```

The SDK MUST:
- Never raise exceptions into host application
- Never delay or block LLM response
- Fail silently if backend unreachable
- Use 2 second timeout on all backend calls, drop silently on timeout

---

## DOCUMENT REFERENCES

Before implementing any component, read the relevant spec:

| What you are building          | Read this first                              |
|--------------------------------|----------------------------------------------|
| Any component                  | docs/specs/af-component-spec-v1.md           |
| Any database model             | docs/specs/af-data-contract-spec-v1.md       |
| Any LLM prompt                 | docs/specs/af-prompt-eval-v1.md              |
| Criteria templates             | docs/specs/af-criteria-template-spec-v1.md   |
| Sprint 1 tasks                 | docs/specs/af-impl-s1-v1.md                  |
| Sprint 2a tasks (attribution)  | docs/specs/af-impl-s2a-v1.md                 |
| Sprint 2b tasks (fix engine)   | docs/specs/af-impl-s2b-v1.md                 |
| Functional requirements        | docs/s1-requirements/af-fr-registry-v1.md    |
| Non-functional requirements    | docs/s1-requirements/af-nfr-registry-v1.md   |
| Acceptance criteria            | docs/s1-requirements/af-ac-registry-v1.md    |
| Gate 1 decisions               | docs/s1-requirements/af-g1-gate-v1.md        |
| Gate 2 checklist               | docs/s1-requirements/af-g2-gate-v1.md        |
| Full roadmap                   | docs/s1-requirements/af-roadmap-v1.md        |

---

## TECH STACK

| Layer       | Technology              | Version  |
|-------------|-------------------------|----------|
| Backend     | FastAPI                 | 0.111.0  |
| Language    | Python                  | 3.12     |
| ORM         | SQLAlchemy (async)      | 2.0.30   |
| Migrations  | Alembic                 | 1.13.1   |
| Database    | PostgreSQL              | 16       |
| Vector      | pgvector                | 0.7+     |
| Cache/Queue | Redis                   | 7        |
| LLM         | Anthropic Claude        | 0.25.0   |
| LLM         | OpenAI                  | 1.30.0   |
| Frontend    | React                   | 18       |
| Styling     | Tailwind CSS            | 3        |
| SDK         | Python (pip)            | 3.12     |

---

## SPRINT SEQUENCE

| Sprint  | Scope                                         | Gate Before Next     |
|---------|-----------------------------------------------|----------------------|
| 1       | Eval Core + Layer 1 + Criteria Templates      | Validate on real agent |
| 2a      | Attribution engine ONLY                       | Gate 3 — manual validation |
| 2b      | Fix engine (after Gate 3)                     | Score delta confirmed |
| 2c      | Layers 2, 2.5, 3                              | Full eval pipeline    |
| 3       | Python SDK + call event ingestion             | Production calls captured |
| 4       | Control dashboard + drift + rollback          | Production control live |

---

## WHAT NOT TO BUILD YET

- Frontend / UI (post Sprint 2c)
- SDK (Sprint 3 — not before)
- Fix engine (Sprint 2b — not before attribution is validated)
- Multi-tenant auth
- Fine-tuning pipelines
- Cross-agent regression (Tier 2 roadmap)
- Adversarial input generation (Tier 3 roadmap)

---

## CURRENT SPRINT

Sprint 1 — Eval Core

Read af-impl-s1-v1.md for the complete Sprint 1 task list.
Gate 1 is APPROVED. Gate 2 required before disk write.
