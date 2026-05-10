# Presentations

Professional slide decks documenting the infrastructure journey from foundation to Conversational AIOps.

---

## Phase 22-23: From Observation to Autonomy ⭐ Latest

### Self-Healing Foundations + The Autonomous Bridge
**File**: phase-22-23-observation-to-autonomy.pdf
**Slides**: 14 slides
**Format**: PDF
**Coverage**: Phases 22–23

The combined-arc story: from observer to actor to autonomous remediator. Phase 22 built the dry-run safety architecture (allowlist → enrichment → simulated action with three-gate safety). Phase 23 elevated dry-run to live execution through a 5-gate pipeline with single-strike rollback.

- **Phase 22 zone**: zero-LLM allowlist matcher → Alertmanager enrichment → dry-run executor (kill-switch + rate-limit + loop guard)
- **Phase 23 spine**: 5 gates in series — kill-switch → rate-limit → pre-flight → restart → stable-duration — with single-strike rollback (any single failure during the 60s window → rolled_back)
- **Three independent safety controls**: kill-switch (operator intent, no audit row), rate-limit (3 actions per metric/node per 10 min, row written), loop guard (structural — every audit row counts toward the rate-limit COUNT)
- **LLM boundary**: zero LLM at any gate; eight consecutive phases without an LLM in the execution path

**Live validation**: searxng restart in production (HTTP 200, 0.4s end-to-end) — the deterministic spine proven against a real Nomad allocation.

**Key Decisions**: ADR-025 (5-gate pipeline rationale), ADR-026 (kill-switch confirmation gate placement), ADR-027 (Three-Layer UI Validation standard).

**Gemini Grade**: A+ × 5 — five consecutive A+ checkpoints across the Phase 22-23 arc (Phase 22: CP-Week-1, CP-Week-2, CP-Phase; Phase 23: CP-Week-1, CP-Week-2). "Deterministic, safe, and verifiable autonomous remediation."

**Use for**:
- Self-healing infrastructure and autonomous remediation discussions
- "How do you safely execute live actions on production?" interviews
- Zero-LLM execution path engineering
- Three-Layer UI Validation methodology (Function-level / AppTest / Manual-smoke triad)

---

## Phase 21: Distributed Foundations ⭐

### Infrastructure Engineering — PostgreSQL Migration + Containerization
**File**: phase-21-distributed-foundations.pdf
**Slides**: 14 slides
**Format**: PDF
**Coverage**: Phase 21

The infrastructure engineering story: Mac Studio removed from the compute plane entirely.

- **Data plane**: SQLite on Mac Studio → PostgreSQL on a cluster node (cross-node writes proven)
- **Compute plane**: raw_exec macOS → docker driver, multi-arch, runs on any cluster node
- **IAM**: postgres SUPERUSER → scoped `app_role` (CRUD only)
- **Containerization**: 0 → 3 services (scraper, lifecycle manager, log event collector)

**The A+ Bar** (four conditions met simultaneously): PostgreSQL on lab-node-3, ≥1000 tests passing,
cross-node writes proven, scraper running on lab-node-1 (not Mac Studio).

**Key Decisions**: `rag/db.py` connection factory, ADR-020 placement policy (architectural-intent
vs operational-state insight), shared-image pattern (one Dockerfile, three entrypoints).

**Gemini Grade**: A+ (CP2) — "A definitive architectural inflection: compute migrated from
development machine to cluster, data migrated to dedicated node, IAM scoped to principle
of least privilege."

**Use for**:
- Distributed systems and data plane migration discussions
- Infrastructure containerization engineering
- "Show me infrastructure work beyond application code" interviews
- PostgreSQL migration patterns (dialect portability, placeholder translation)

---

## Observability Trilogy — Phases 18–20

### Predict → Explain → Remember & Verify
**File**: Observability-Trilogy.pdf
**Slides**: 14 slides
**Format**: PDF
**Coverage**: Phases 18–20

The three-phase Conversational AIOps arc — a single coherent story told across nine weeks of engineering:

- **Phase 18 (Predict)**: Zero-LLM OLS regression engine, ~0.05ms P95, Time-to-Threshold
  forecasting, observer effect, Glass Box Tab 7
- **Phase 19 (Explain)**: Cross-modal correlation engine, Nomad/Consul event collection,
  multiplicative scoring, four-state diagnostic UI (Candidate / Unknown Cause /
  No Events Logged / Collector Gap), <10ms Top Cause answer
- **Phase 20 (Remember & Verify)**: DiagnosticContext with bounded context window guard
  (≤200 chars, dual-layer truncation), Verifiable Inference (live trend on follow-up),
  Tab 8 Diagnostic Thread (append-only session investigation timeline),
  heartbeat semantic contract, unified lifecycle manager

**The Demo**: Ask "Why is CPU high?" — answered in <10ms. Ask "Is that still related to
the restart?" — the system remembers the diagnosis, checks the live trend, and tells you
whether the original conclusion still holds.

**Gemini Concepts Coined**: Verifiable Inference, Diagnostic Thread, Pivot Event,
Human Deliberation Buffer, Observability Tempo

**Gemini Grade**: A+ (CP3) — "A definitive shift from building a tool to engineering a system"

**Speaker notes embedded** — delivery guide with per-slide opening lines, transitions,
audience pivots (recruiter / senior engineer / DevOps), and timing guide (~20 min)

**Use for**:
- AIOps and conversational AI design discussions
- Zero-LLM deterministic analytics demonstration
- Production observability system design
- "Show me your latest work" interview conversations

---

## Phase 9: Agentic RAG with Glass Box AI ⭐

### From Library to Co-Pilot
**File**: Phase-9-Presentation-Agentic-RAG.pdf
**Slides**: 14 slides
**Format**: PDF
**Coverage**: Phase 9 (Agentic RAG)

The intelligent co-pilot transformation story:
- **Week 1**: MCP Tools Foundation (5 operational tools, ADR Threshold Injector)
- **Week 2**: Agentic Reasoning (98.2% intent accuracy, 100% multi-step success)
- **Week 3**: Glass Box AI (visual transparency with 3-tab interface) ⭐

**Highlights**:
- 5 MCP tools: Prometheus, Nomad, Consul APIs with security guardrails
- Intent classification: 98.2% accuracy, <1ms response
- Glass Box AI Innovation: Thought Trace, Timeline, Performance dashboards
- Revealed 99.9% of execution time in MCP API calls (not AI reasoning)

**Key Achievement**: 98.7% overall accuracy, 21-121ms (41-238× faster than target),
190 tests passing

**Use for**: Agentic AI, Glass Box explainability, systematic engineering interviews

---

## Phase 10: Performance & Intelligence Optimization ⭐

### From Unreliable to Production-Ready AI
**File**: Phase 10 - RAG Performance and Intelligence.pdf
**Slides**: 14 slides
**Format**: PDF
**Coverage**: Phase 10 (Performance & Intelligence)

- **Week 1**: Infrastructure performance (605,000× cached speedup)
- **Week 2**: Chain of Verification — 3ms overhead, 7-component tiered system
- **Week 3**: Confidence Paradox diagnostic (51pp improvement from 50 lines)

**Key Achievement**: 18% fallback (51pp ↓), 605,000× speedup, 3ms CoVe, 416 tests

**Use for**: Performance optimization, AI reliability engineering, root cause analysis

---

## Phase 8: Retrieval Quality Optimization

### The 90% Accuracy Journey
**File**: phase-8-optimization-journey.pdf
**Slides**: 13 slides
**Format**: PDF
**Coverage**: Phase 8 (Retrieval Optimization)

The breakthrough optimization story — 0% → 90% accuracy through three-layer systematic
optimization. Session 2 delivered +55pp from document type boosting alone.

**Key Achievement**: 0% → 90% accuracy, exceeded 60-70% target by 20-30pp, 7.5 hours

**Use for**: Systematic optimization, data-driven engineering, interview storytelling

---

## Phase 7: RAG Platform Foundation

### Production Knowledge System
**File**: phase-7-rag-platform.pdf
**Slides**: 15 slides
**Format**: PDF
**Coverage**: Phase 7 (Advanced RAG Platform)

RAG platform creation: 85 documents, 611 chunks, 5-layer architecture, 200× cache speedup.
Discovery of the "Retrieval Ceiling" sets up the Phase 8 optimization story.

**Use for**: RAG platform architecture, Phase 8 context, engineering maturity

---

## Infrastructure Overview (Phases 4-6)

### Foundation & Production Operations
**File**: infrastructure-overview.pdf
**Slides**: 12 slides
**Format**: PDF
**Coverage**: Phases 4-6

Nomad cluster, Consul service mesh, Prometheus + Grafana observability,
hybrid Ollama deployment (24× GPU speedup). The foundation that everything else builds on.

**Use for**: General technical overview, infrastructure discussions

---

## Story Arc

**Phase Progression**:
1. **Infrastructure** (Phases 4-6): Foundation and production operations
2. **Phase 7**: Production RAG platform (85 docs, 611 chunks)
3. **Phase 8**: Systematic optimization (0% → 90% accuracy) ⭐
4. **Phase 9**: Intelligent co-pilot transformation (Glass Box AI) ⭐
5. **Phase 10**: Production-ready AI (18% fallback, 605,000× speedup) ⭐
6. **Observability Trilogy** (Phases 18–20): Predict → Explain → Remember & Verify ⭐
7. **Methodology Journey** (Phases 1–20): How the discipline to build it all was engineered ⭐

**Interview Narrative**: *"We built solid infrastructure, created a production RAG platform,
optimized it to 90% accuracy, transformed it into an intelligent co-pilot with Glass Box AI,
then built a Conversational AIOps system that predicts metric trends, explains anomalies,
and maintains the diagnostic thread across conversation turns. And if you want to understand
how that happened systematically — that's a separate presentation."*

---

## Coverage Summary

| Presentation | Phases | Slides | Key Achievement |
|---|---|---|---|
| **Infrastructure Overview** | 4-6 | 12 | Nomad + Consul + Prometheus, 24× GPU speedup |
| **Phase 7** | 7 | 15 | Production RAG, 5-layer architecture, retrieval ceiling |
| **Phase 8** | 8 | 13 | 0% → 90% accuracy, three-layer boosting |
| **Phase 9** | 9 | 14 | 98.7% agentic accuracy, Glass Box AI ⭐ |
| **Phase 10** | 10 | 14 | 18% fallback, 605,000× speedup, CoVe ⭐ |
| **Phase 17** | 17 | 14 | 0.208ms conflict resolution, Glass Box 6 tabs ⭐ |
| **Phase 21** | 21 | 14 | PostgreSQL migration, 3 containerized, cross-node writes, A+ ⭐ |
| **Phase 22-23** | 22–23 | 14 | 5-gate live execution, three independent safety controls, zero LLM in execution path, A+ × 5 ⭐ |
| **Observability Trilogy** | 18–20 | 14 | Predict→Explain→Remember, Verifiable Inference ⭐ |
| **Methodology Journey** | 1–20 | 12 | 7 practices, 35 weeks, crawl → walk → craft ⭐ |
| **Total** | 1–23 | **136** | Complete journey: infrastructure → Autonomous Bridge |

---

## Presentation Philosophy

- **Visual Clarity**: Architecture diagrams at multiple zoom levels
- **Data-Driven**: Metrics and results prominently featured
- **Systematic Approach**: Methodical problem-solving showcased
- **Engineering Honesty**: Problems positioned as discoveries
- **Professional Quality**: Interview and presentation-ready
- **Innovation Highlight**: Unique contributions clearly demonstrated

---

**Status**: 10 presentations complete ✅
**Total Slides**: 136 (current)
**Latest**: Phase 22-23 — From Observation to Autonomy (May 2026) ⭐
**Zero-LLM streak**: 8 consecutive phases (16–23)
