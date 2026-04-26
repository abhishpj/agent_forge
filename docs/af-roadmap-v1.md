# af-roadmap-v1
# Project: AgentForge
# Document Type: Full Product Roadmap
# Version: 1.0
# Status: Approved
# Last Updated: 2026-04

---

## PRODUCT DEFINITION

AgentForge is a CI/CD + observability + control system for non-deterministic AI behaviour.

It brings the same discipline software teams apply to code:
- Unit tests → Eval suites
- CI pipeline → Deployment gate
- Staging environment → Eval core
- Production monitoring → SDK + drift detection
- Rollback → One-click version rollback

---

## SINGLE POINT OF FAILURE

> If evaluation + failure attribution is weak, everything collapses.

The entire roadmap is sequenced to protect against this.
Attribution is validated manually before fix is built.
Fix is not built until attribution is proven correct.

---

## CORRECTED EXECUTION SEQUENCE

```
Sprint 1   →  Layer 1 + API + DB + Criteria Templates
              Validate on real agent
              ↓
Sprint 2a  →  Attribution engine ONLY
              Gate 3 — manual validation (min 3 runs, 2 correct)
              ↓
Sprint 2b  →  Fix engine (ONLY after Gate 3)
              ↓
Sprint 2c  →  Layers 2, 2.5, 3
              ↓
Sprint 3   →  SDK (near-zero integration, safe by default)
              ↓
Sprint 4   →  Control dashboard
```

---

## SPRINT 1 — EVAL CORE (Weeks 1–2)

**Goal:** Working eval engine that scores any agent output against user-defined criteria.
**Sellable as:** Standalone eval tool before any other sprint.

### Core Deliverables
- Agent, Prompt, TestSuite, EvalRun APIs
- Layer 1 deterministic validators (schema, field, string, regex, length)
- Deployment gate enforcement
- Prompt version management

### Mandatory Additions (from research)
| Feature | Why Mandatory |
|---|---|
| Trajectory validator | Agents using wrong tool order look like they pass but are broken |
| Tool selection precision | Wrong tool = wrong result even if output looks correct |
| Operating envelope validators (cost + latency) | Enterprises require budget and SLA enforcement |
| Faithfulness constraint in LLM judge | Without it, judge misses hallucinations entirely |

### Criteria Template Layer (onboarding critical)
- 15+ pre-built validator templates across domains
- Domain starter packs: QA automation, RAG, General purpose
- GET /criteria-templates, POST /domain-packs/{id}/apply
- Without this layer onboarding friction kills adoption

### Gate Before Sprint 2a
- Sprint 1 validated on at least one real agent (Skynet or TA tool)
- Layer 1 catches real failures on real inputs
- Human reviewer confirms eval results make sense

---

## SPRINT 2a — ATTRIBUTION ENGINE (Week 3)

**Goal:** Given a failing eval run, identify exactly which prompt module caused it.
**Critical:** This sprint ends with Gate 3 — manual validation of attribution accuracy.

### Deliverables
- Perturbation loop — remove one module at a time, measure score delta
- Ranked module blame — highest negative delta = primary contributor
- LLM attribution assist — semantic hints (supplementary, not authoritative)
- FailureTrace persistence
- Human review interface for attribution validation

### Gate 3 — Non-Negotiable
- Minimum 3 real agent failures attributed
- Attribution correct in at least 2 of 3 (human verdict)
- Attribution confidence > 0.7 for correct attributions
- Fix engine returns 423 Locked until Gate 3 satisfied

---

## SPRINT 2b — FIX ENGINE (Week 4)

**Goal:** Given an attributed failure, rewrite only the failing module and verify the fix.
**Prerequisite:** Gate 3 must be approved.

### Deliverables
- Fix generator — rewrites only the attributed module
- Module isolation — all other modules byte-for-byte identical
- Pairwise comparison judge — which output is better, not just score delta
- Fix accepted only if: score delta >= threshold AND pairwise selects fixed
- Auto vs manual trigger mode per agent config
- Human review flagged when score and pairwise disagree

---

## SPRINT 2c — FULL EVAL STACK (Week 5)

**Goal:** Complete all four validation layers.

### Layer 2 — Statistical
- Semantic similarity scorer (pgvector cosine)
- JSON edit distance scorer
- Output consistency scorer
- RAGAS faithfulness metric
- RAGAS context precision
- RAGAS context recall
- RAGAS answer correctness

### Layer 2.5 — Reasoning Signals (NON-BLOCKING)
- Stability scorer — rephrase input N times, measure variance
- Invariant checker — user-defined logical invariants
- Reasoning critique — LLM identifies violations only
- Combined reasoning score: 0.5 * stability + 0.3 * constraint + 0.2 * critique
- NEVER a hard gate — informational only in V1

### Layer 3 — LLM Judge (constrained hard)
- Single judge mode (Claude or OpenAI per suite)
- Ensemble judge mode — N judges, median score (reduces noise)
- Reference-free mode — score without expected output
- Judge calibration mode — measure judge variance
- Faithfulness always evaluated (FR-021)

---

## SPRINT 3 — SDK (Week 6)

**Goal:** Intercept every production LLM call. Own the execution path.
**Positioning:** "Near-zero integration, safe by default" — NOT "zero config"

### Deliverables
- pip install agentforge
- agentforge.init(api_key="...")
- agentforge.complete() — wraps Claude and OpenAI identically
- Async capture: version, input, output, latency, cost, session_id
- POST /sdk/ingest — 202 immediately, async processing
- LLMCallEvent append-only table
- Async eval trigger on every ingested call
- Silent failure mode — SDK NEVER breaks host application
- SDK overhead: max 5ms, never blocks LLM response

### Production Failure Auto-Promotion
- Failing production calls flagged in dashboard
- One-click: promote to test suite as new test case
- Auto-promotion mode: configurable threshold

---

## SPRINT 4 — CONTROL DASHBOARD (Week 7)

**Goal:** Full production visibility and one-click control.

### Priority Order (must-have first)
1. Failure rate per prompt version
2. Drift detection + alerts
3. Rollback control

### Then
4. Quality score trend over time
5. Top failing input patterns (clustered)
6. Cost and latency trends
7. Version comparison / diff view
8. Multi-turn simulation eval

---

## TIER 1 — HIGH VALUE POST-V1

| Feature | Description | Why |
|---|---|---|
| Judge calibration | Measure LLM judge variance, flag inconsistent judges | Eval scores only trustworthy if judge is consistent |
| Auto-regression builder | Suggest new test cases from production failure patterns | Test suite improves itself |
| Batch production scoring | Daily quality report from last 24h production calls | Morning digest without manual work |
| Criteria template library growth | Add new domain packs: customer support, code gen, finance | Key acquisition driver |

---

## TIER 2 — LARGER SCOPE ROADMAP

| Feature | Description |
|---|---|
| Component-wise RAG deep eval | Separate retriever vs generator evaluation |
| Full RAGAS integration | All 4 RAGAS metrics as Layer 2 validators |
| Dynamic ground truth | Callable expected output for live data agents |
| Autonomy-level aware gating | Stricter gates for fully autonomous agents |

---

## TIER 3 — ADVANCED / RESEARCH-GRADE

| Feature | Description |
|---|---|
| Full multi-turn simulation | Red team library with adversarial personas |
| Fine-tuning dataset export | Passing eval runs as labelled training data |
| Cross-agent regression | When one agent changes, test all dependent agents |
| Adversarial input generation | Auto-generate edge cases to find failure modes |

---

## WHAT NOT TO BUILD (YET)

- Frontend canvas / drag-and-drop UI
- Agent execution / deployment runtime
- Multi-tenant auth (V1 = single org)
- Fine-tuning pipelines
- Multi-model optimisation UI

---

## FEATURE COUNT SUMMARY

| Sprint / Tier | Features | Status |
|---|---|---|
| Sprint 1 | 14 tasks + 4 mandatory + criteria templates | In progress |
| Sprint 2a | 7 tasks (attribution) | Planned |
| Sprint 2b | 7 tasks (fix engine) | Planned (post Gate 3) |
| Sprint 2c | 15 features (layers 2, 2.5, 3) | Planned |
| Sprint 3 | 8 features (SDK) | Planned |
| Sprint 4 | 8 features (dashboard) | Planned |
| Tier 1 | 4 features | Post-V1 |
| Tier 2 | 4 features | Roadmap |
| Tier 3 | 4 features | Future |
| TOTAL | 75+ features | Full roadmap |

---

## THE COMPOUNDING MOAT

The real long-term moat is not the SDK. It is this loop:

```
Production call fails
        ↓
Auto-flagged in dashboard
        ↓
Promoted to test suite with one click
        ↓
Test suite now prevents that failure recurring
        ↓
Next time that input pattern appears → caught before deploy
        ↓
Over time: test suite grows from real failures
Over time: platform gets harder to replace
Over time: switching cost compounds
```

> "Failures become test cases. Test cases prevent regressions."
> That is the compounding quality loop. That is the moat.
