# af-g2-gate-v1
# Project: AgentForge
# Document Type: Gate 2 — Final Approval Before Disk Write
# Version: 1.0
# Status: Draft
# Last Updated: 2026-04

---

## PURPOSE

Gate 2 is the second human approval checkpoint in the AgentForge ADF pipeline.
It occurs after the coding agent has produced all code and file content for review,
but BEFORE anything is written to disk.

Gate 2 ensures that a human has reviewed and approved all generated content.

---

## GATE 2 RULES

1. Coding agent presents all generated code and files for human review
2. Human reviews all content before any disk write occurs
3. Human may request changes — agent revises and re-presents
4. Only after explicit human approval does the agent write to disk
5. Gate 2 is recorded with approver name and timestamp per sprint

---

## GATE 2 CHECKLIST

### Code Quality
- [ ] All functions are 20 lines or fewer
- [ ] All functions have docstrings
- [ ] No abbreviations in variable names
- [ ] No nested ternaries or one-liners for complex logic
- [ ] No magic numbers — all constants named
- [ ] One responsibility per function

### Architecture Compliance
- [ ] Folder structure matches af-component-spec-v1
- [ ] All imports resolve correctly
- [ ] No circular dependencies
- [ ] Database models match af-data-contract-spec-v1
- [ ] API endpoints match af-fr-registry-v1 specifications

### Requirement Traceability
- [ ] Every implemented feature traces to a FR in af-fr-registry-v1
- [ ] No features implemented that are not in the FR registry
- [ ] All P0 requirements for this sprint are implemented

### Test Coverage
- [ ] Unit tests present for all Layer 1 validators
- [ ] Integration tests present for all P0 API endpoints
- [ ] All tests pass before approval

### Security
- [ ] No API keys or secrets in source code
- [ ] All endpoints require authentication
- [ ] No raw SQL queries (ORM only)

---

## GATE 2 APPROVAL

| Field        | Value          |
|--------------|----------------|
| Sprint       | _______________|
| Reviewed By  | _______________|
| Date         | _______________|
| Changes Requested | _________|
| Final Status | PENDING        |

---

## POST-APPROVAL ACTIONS

Once Gate 2 is approved, the coding agent is authorised to:
1. Write all approved files to disk
2. Run database migrations
3. Execute tests to confirm all pass

The coding agent is NOT authorised to:
- Modify any file after disk write without a new Gate 2 review
- Write files not included in the approved content
- Skip any checklist item
