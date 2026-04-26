# af-g1-gate-v1
# Project: AgentForge
# Document Type: Gate 1 — Ambiguity Resolution Gate
# Version: 1.0
# Status: APPROVED
# Last Updated: 2026-04

---

## GATE 1 STATUS: APPROVED

All open questions resolved. All architectural decisions confirmed.
Sprint 1 coding is authorised to begin.

---

## RESOLVED DECISIONS

| ID  | Question                              | Decision                                  |
|-----|---------------------------------------|-------------------------------------------|
| Q1  | Raw prompt auto-split strategy        | Hybrid — heuristic first, LLM refines     |
| Q2  | Layer execution mode                  | Configurable per test suite               |
| Q3  | Sync vs async eval                    | Configurable timeout                      |
| Q4  | Fix generation trigger                | Configurable per agent                    |
| Q5  | Reasoning signal gate behaviour       | User configurable per test suite          |
| Q6  | LLM judge model support               | Claude + OpenAI in V1                     |

---

## CONFIRMED ARCHITECTURAL DECISIONS

| ID     | Decision                                                              |
|--------|-----------------------------------------------------------------------|
| AD-001 | FastAPI (Python 3.12)                                                 |
| AD-002 | PostgreSQL 16 with pgvector                                           |
| AD-003 | Redis 7 for session cache and async task queue                        |
| AD-004 | FastAPI BackgroundTasks for MVP                                        |
| AD-005 | OpenAI text-embedding-3-small for Layer 2 similarity scoring          |
| AD-006 | React 18 with Tailwind CSS                                            |
| AD-007 | Python SDK first (pip installable)                                    |
| AD-008 | Max readability; functions ≤ 20 lines; full variable names; docstrings|

---

## GATE 1 APPROVAL

| Field       | Value          |
|-------------|----------------|
| Approved By | Abhish         |
| Date        | 2026-04-25     |
| Status      | APPROVED       |
