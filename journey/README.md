# Journey

Phase-by-phase narratives documenting the evolution of the HomeLab infrastructure platform.

---

## Story Coverage

### **Detailed Journey Narratives**

### [Phase 23 — The Autonomous Bridge](phase-23-autonomous-bridge.md) ✅ ⭐ NEW
**Focus**: Live execution with three independent safety controls, zero LLM in the gate-and-restart path, single-strike rollback. Five consecutive A+ Gemini checkpoints across the Phase 22-23 arc.
**Achievement**: Observer → actor → autonomous remediator. The kill-switch (operator intent, no audit row) + rate-limit (automatic safety, row written) + loop guard (structural — rate-limit SQL has no outcome filter) are three orthogonal failure modes that each independently halt execution. The 5-gate pipeline shipped without LLM dependency at any gate, validated against a live searxng restart (HTTP 200, 0.4s).
**Key Innovations**: 5-gate live executor (`rag/agent_executor.py::execute_live`), ExternalID + CreateTime stale-alloc detection (catches Nomad-rescheduled allocs mid-window), kill-switch confirmation gate relocation (the if-prompt rerun-disappearing bug surfaced by manual smoke + fixed by architectural placement, ADR-026), Three-Layer UI Validation standard codified (ADR-027 — Function-level / AppTest / Manual-smoke triad).
**Metrics**: 1181 tests collected (+73 from Phase 22), 1179 passing (+92), 0 failing, 19 Prometheus skip-debt closed, zero-LLM streak 8 phases (16-23), zero-regression streak 8 phases.
**Grade**: A+ (Gemini CP1 Week 1 + CP2 Week 2 — consecutive A+) | April 2026

---

### [Phase 22 — Self-Healing Foundations](phase-22-self-healing-foundations.md) ✅ ⭐
**Focus**: Zero-LLM self-healing — allowlist-driven suggestions, dry-run executor with three-gate safety, 8/8 Glass Box tabs under automated regression harness
**Achievement**: Observer → actor transition without betting the farm. Three-step arc (observe → suggest → simulate) gated by kill-switch + rate-limit + loop guard. The kill-switch (operator intent) writes NO audit row; the rate-limit (automatic safety) writes a row with `outcome="rate_limited"`; the loop guard is structural — the rate-limit SQL has no outcome filter, so the agent's own dry-run rows count toward its budget. First zero-failures run since Phase 21 Consul drift: 1087 passing / 0 failed.
**Key Innovations**: `rag/agent_actions.py` (zero-LLM allowlist lookup — unmatched Top Cause returns `None`, not an LLM fallback), `rag/agent_executor.py` (three-gate safety, `dry_run=TRUE` always, no HTTP/subprocess imports), 25-test UI harness via `streamlit.testing.v1.AppTest` with cache-hit shortcut and session-state pre-seeding, Testability Principles codified mid-phase (return plain data, no side effects in return values) enabling 20-minute module delivery in Session 5.
**Metrics**: 1108 tests (+72 net), 1087 passing (+85), 0 failing (−15), 25 UI harness tests @ 0% flakiness, CI green on ubuntu-latest / Python 3.12, zero-LLM streak 7 phases, A+ grade.
**Grade**: A+ (Gemini CP-Week-1 / CP-Week-2 / CP-Phase all confirmed) | April 2026

---

### [Phase 21 — Distributed Foundations](phase-21-distributed-foundations.md) ✅
**Focus**: PostgreSQL migration, containerized observability, distributed compute plane
**Achievement**: Mac Studio removed from the compute plane entirely. Three services containerized (docker driver, multi-arch), data plane migrated from SQLite to PostgreSQL on a separate cluster node, scoped IAM role replacing SUPERUSER. Cross-node writes proven: scraper on lab-node-1 writing to PostgreSQL on lab-node-3.
**Key Innovations**: `rag/db.py` connection factory (PostgreSQL ↔ SQLite, `%s` placeholder translation), `rag/alerting.py` Alertmanager webhook enrichment, ADR-020 Nomad-native discovery + job-file placement policy (operational-state vs architectural-intent insight), shared-image pattern (one Dockerfile, three entrypoints), scoped `app_role` replacing SUPERUSER.
**Metrics**: 1036 tests (1002 passing), 37 new tests, zero regressions, zero-LLM streak 6 phases, A+ grade.
**Grade**: A+ (Gemini CP2 Confirmed) | April 2026

---

### [Phase 20 — Reliability & Resonance](phase-20-reliability-resonance.md) ✅
**Focus**: Conversational AIOps — memory, verification, Diagnostic Thread
**Achievement**: The system now maintains the diagnostic thread over time. Ask "Why is CPU high?" — it correlates and answers. Ask "Is that still related to the restart?" — it remembers the diagnosis, checks the live trend, and tells you whether the original conclusion still holds. Closes the three-phase Observability Trilogy arc: predict (Phase 18) → explain (Phase 19) → remember & verify (Phase 20).
**Key Innovations**: DiagnosticContext with bounded context window guard (≤200 chars, dual-layer truncation), follow-up detection with new-query override, Verifiable Inference (live TrendMonitor status appended to cached responses), Tab 8 Diagnostic Thread (append-only session investigation timeline), Heartbeat semantic contract (ok-heartbeat resolves "Quiet Infrastructure Ambiguity"), Unified lifecycle manager (atomic 14-day/7-day retention via Nomad periodic job).
**Metrics**: 999 tests, 980/980 runnable @ 100%, zero regressions, 3 new modules, 9 Gemini mandates, zero-LLM streak 5 phases.
**Grade**: A+ (Gemini CP3 Validated) | April 2026

---

### [Phase 19 — Cross-Modal Diagnostic Correlation](phase-19-cross-modal-correlation.md) ✅
**Focus**: Zero-LLM correlation engine, four-state Glass Box Tab 7, end-to-end diagnostic query response
**Achievement**: Metric inflections × Nomad/Consul events → Top Cause answer in <10ms. "Why is CPU high?" → "Likely caused by: searxng (restart) 45s ago [score=0.46]." Multiplicative scoring with suppression threshold. Controlled negative validation (Scenario 3). Four-state Quiet Period Paradox resolution.
**Grade**: A+ (Gemini Validated) | April 2026

---

### [Phase 18 — Predictive Diagnostics](phase-18-predictive-diagnostics.md) ✅
**Focus**: Zero-LLM predictive engine, Glass Box Tab 7, observer effect
**Achievement**: OLS regression forecasting 9 Prometheus series at ~0.05ms P95 (100× under 5ms budget). Glass Box Tab 7 with R² color-coding and self-aware observer effect diagnostics.
**Grade**: A+ (Gemini Validated) | March 2026

---

### [Phase 17 — Advanced Synthesis](phase-17-advanced-synthesis.md) ✅
**Focus**: Conflict detection, resolution, and multi-document synthesis with full provenance
**Achievement**: 0.208ms heuristic engine resolving 7 conflict categories, zero LLM dependencies
**Grade**: A+ (Gemini Validated) | March 2026

---

**Phase 16: Analytical Intelligence & Operational Guardrails** ⭐
**File**: [Phase-16-Journey-Narrative.md](Phase-16-Journey-Narrative.md)
**Summary**: Self-aware analytical system, 100% anomaly detection, temporal coreference, "top 1%" sustained

The analytical intelligence story of transforming a high-performance RAG system into a self-aware analytical system that monitors, diagnoses, and analyzes its own performance evolution proactively. Features conversational memory optimization, EAV metric snapshots with hybrid anomaly detection, adaptive sliding windows, and temporal coreference resolution with entity pinning.

**Highlights**:
- Self-aware system: Monitors own performance evolution via metric snapshots
- Conversational memory: 7.05ms overhead (86% under 50ms target), 100% cache hit
- Anomaly detection: 100% accuracy (hybrid two-stage), zero false positives
- Adaptive sliding window: <1ms overhead, 7.4% token usage (50% headroom)
- Temporal coreference: 100% accuracy (9/9 "Hard" tests)
- Test growth: +87 tests (+18.7%), 552 total, zero regressions
- External validation: Sustained A+ (weeks 2-3), "top 1%" recognition maintained

**Key Metrics**: 100% anomaly detection, 100% coreference, 7.4% token usage, 552 tests @ 100%, Grade A+

---

**Phase 15: Query Decomposition Robustness & Latency Optimization** ⭐
**File**: [phase-15-journey.md](phase-15-journey.md)
**Summary**: 100% recall, 4.22s P95, conversational memory, "top 1%" recognition

The methodology validation story of achieving elite-tier RAG performance through hypothesis-driven planning, external validation checkpoints, and data-driven pivots.

**Highlights**:
- 100% recall achievement (+15pp from Phase 14, zero false negatives)
- P95 latency: 6.35s → 4.22s (-33.5%, stretch target exceeded)
- Conversational memory: 100% coreference accuracy, 91.7% multi-turn success
- RAGAS validation: 1.000 Context Precision (perfect), 0.861 Faithfulness
- External recognition: "Top 1% of local RAG implementations" (Gemini)
- Methodology validation: "Benchmark → Experiment → Validate → Pivot" (Gemini-recognized)

**Key Metrics**: 100% recall, 4.22s P95, 100% coreference, 91.7% multi-turn, Grade A+ / A+ / A+

---

**Phase 13+14: LLM-Guided Query Decomposition** ⭐
**File**: [phase-13-14-evolution.md](phase-13-14-evolution.md)
**Summary**: Pattern-based to LLM-guided transformation, 4.4× GOOD rate improvement

**Highlights**:
- 4.4× GOOD rate improvement (15% → 66.7%)
- 2× precision improvement (50% → 100%, zero POOR decompositions)
- 7 intent types operational
- MVP validation: Week 2 delivered 74% of total improvement

**Key Metrics**: 66.7% GOOD rate, 100% precision, 7 intent types, 3.95s latency, Grade A+

---

**Phase 12: Data-Driven Retrieval Optimization**
**File**: [phase-12-retrieval-optimization.md](phase-12-retrieval-optimization.md)
**Summary**: From 72% to 92% hit rate through strategic corpus expansion and empirical validation

**Highlights**:
- 30-minute Session 1 investigation → strategic pivot (4-6× ROI)
- Corpus gap discovery: 43% of failures were missing data, not retrieval quality
- Test discovery: 469 tests found (was only testing 17, missed 96%)
- Bottleneck shift: Retrieval optimized → routing classification exposed

**Key Metrics**: 92% hit rate, 2,095 chunks (3.4× growth), 469 tests, 0 regressions, Grade A

---

**Phase 10: Performance & Intelligence Optimization** ⭐
**File**: [phase-10-performance-optimization.md](phase-10-performance-optimization.md)
**Summary**: From 69% to 18% fallback with 605,000× speedup

**Highlights**:
- 69% → 18% fallback rate journey (51pp improvement)
- Chain of Verification (CoVe) 7-component architecture
- 605,000× cached query speedup
- Confidence Paradox diagnostic (high confidence = worse results?!)
- Surgical fix: 50 lines, 51pp improvement, 12 minutes

**Key Metrics**: 18% fallback, 3ms CoVe overhead, 3 weeks, 15 sessions

---

**Phase 8: Retrieval Quality Optimization** ⭐ **Featured**
**File**: [phase-8-optimization.md](phase-8-optimization.md)
**Summary**: From 0% to 90% accuracy in 7.5 hours

**Highlights**:
- 0% → 90% search accuracy journey
- Three-layer boosting architecture (Hybrid Search + Document Type + Topic Authority)
- Data-driven decision making (boost factors calculated, not guessed)

**Key Metrics**: 90% accuracy, ~25ms latency, 8 sessions, ~7.5 hours

---

**Phase 9: Agentic RAG with Glass Box AI** ⭐
**File**: [phase-9-agentic-rag.md](phase-9-agentic-rag.md)
**Summary**: From static library to intelligent co-pilot in 3 weeks

**Highlights**:
- 5 MCP tools operational (Prometheus, Nomad, Consul APIs)
- 98.7% agentic accuracy (intent + reasoning + retrieval)
- Glass Box AI: Visual transparency (3-tab visualization)
- Response time: 21-121ms (41-238× faster than target)

**Key Metrics**: 98.7% accuracy, 3 weeks (11 sessions), ~9 hours, 190 tests

---

### **Foundation Documented in Other Formats**

The complete technical journey (Phases 4-9) is also documented across presentations, ADRs, and screenshots. Rather than duplicate content, we provide deep coverage of the most compelling stories.

---

## Foundation References

### Phases 4-6: Infrastructure & Observability
**Covered in**: [Infrastructure Overview Presentation](../presentations/infrastructure-overview.pdf) (12 slides)  
**Key Decision**: [ADR-001: Orchestration Platform Selection](../architecture/decisions/ADR-001-orchestration-platform.md)

### Phase 7: Advanced RAG Platform
**Covered in**: [Phase 7 RAG Platform Presentation](../presentations/phase-7-rag-platform.pdf) (15 slides)  
**Key Decision**: [ADR-008: Advanced RAG Platform](../architecture/decisions/ADR-008-advanced-rag-platform.md)

### Phase 8: Retrieval Optimization (Featured Story)
**Complete Story**: [Phase 8 Journey Narrative](phase-8-optimization.md) + [ADR-009](../architecture/decisions/ADR-009-retrieval-optimization.md) + [Detailed Metrics](../results/phase-8-metrics.md)

### Phase 9: Agentic RAG (Complete)
**Complete Story**: [Phase 9 Journey Narrative](phase-9-agentic-rag.md) + [ADR-010](../architecture/decisions/ADR-010-Agentic-RAG-Architecture.md) + [Architecture Diagrams](../architecture/diagrams/README.md#agentic-rag-system-architecture-phase-9)

---

## Storytelling Philosophy

**Quality Over Quantity**: Deep coverage of breakthrough stories, visual coverage for foundation phases.

**Interview Navigation**:
- "Tell me about a challenging project" → Phase 8 (0% → 90% transformation)
- "Show me AI/ML engineering work" → Phase 9 (agentic RAG + Glass Box AI)
- "Tell me about your engineering methodology" → Phase 15 ("top 1%" + hypothesis-driven)
- "How do you build observable/self-aware systems" → Phase 16 (anomaly detection, entity tracking)
- "Show me AIOps / predictive systems" → Phase 18 (OLS regression, observer effect)
- "How do you approach conversational AI design" → Phase 20 (Verifiable Inference, Diagnostic Thread)
- "How do you approach system design" → ADR-010, ADR-014, ADR-016

---

## Story Arc: Foundation → Build → Optimize → Evolve → Observability Trilogy

**Foundation** (Phases 4-6): Nomad cluster, Consul, Prometheus, hybrid Ollama  
**Build** (Phase 7): Production RAG platform  
**Optimize** (Phase 8): ⭐ 0% → 90% accuracy  
**Evolve** (Phase 9): ⭐ Agentic co-pilot + Glass Box AI  
**Excellence** (Phases 10-16): ⭐ Performance → Retrieval → LLM Intelligence → Memory → Self-Awareness  
**Observability Trilogy** (Phases 17-20): Synthesis → Prediction → Explanation → Memory & Verification ⭐

> **Phase 18**: The system predicts the future.  
> **Phase 19**: The system explains the past.  
> **Phase 20**: The system maintains the diagnostic thread over time.

---

**Status**: Phases 8, 9, 10, 12, 13+14, 15, 16, 17, 18, 19, 20, 21, 22, 23 journeys complete  
**Latest**: Phase 23 — The Autonomous Bridge (5-gate live execution, three independent safety controls, zero LLM in execution path, A+)  
**Zero-LLM streak**: 8 consecutive phases (16–23)  
**Test coverage**: 1181 tests, 1179 passing, 0 failing  
**Philosophy**: Deep dive on breakthrough stories + presentations for context
