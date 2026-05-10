# Architecture

Architecture decisions, system diagrams, and technology choices.

---

## Architecture Decision Records (ADRs)

### ADR-001: Orchestration Platform Selection
**File**: [decisions/ADR-001-orchestration-platform.md](decisions/ADR-001-orchestration-platform.md)  
**Date**: December 15, 2025  
**Status**: ✅ Approved

Infrastructure foundation decision: HashiCorp Nomad vs Kubernetes vs Docker Swarm.

**Key Decision**: HashiCorp Nomad chosen for optimal balance of simplicity (9/10), resource efficiency (~200MB overhead), and learning value in homelab environment.

**Weighted Score**: 7.95 (vs K3s: 7.0, Swarm: 7.45)

**Why**: Operational simplicity prioritized over enterprise pattern mimicry. Enables focus on workload experimentation rather than platform maintenance.

---

### ADR-008: Phase 7 Advanced RAG Knowledge Platform
**File**: [decisions/ADR-008-advanced-rag-platform.md](decisions/ADR-008-advanced-rag-platform.md)  
**Date**: January 2-3, 2026  
**Status**: ✅ Complete

Production RAG platform with conversation memory, web UI, and 4-tier caching.

**Key Decisions**:
- JSON embeddings over vector database (simple, adequate for 85 docs)
- SQLite WAL mode for conversation persistence
- 4-tier caching architecture (200× speedup, 10ms cache hits)
- Streamlit web UI on Nomad cluster

**Results**: 85 documents, 100% accuracy baseline, foundation for Phase 8 optimization

---

### ADR-009: Phase 8 Retrieval Quality Optimization
**File**: [decisions/ADR-009-retrieval-optimization.md](decisions/ADR-009-retrieval-optimization.md)  
**Date**: January 5-6, 2026  
**Status**: ✅ Complete

Systematic optimization achieving 90% accuracy through three-layer boosting architecture.

**Key Decisions**:
- Hybrid search with α=0.3 ratio (BM25 30% + Vector 70%)
- Document authority boosting (ADR=1.5×, Sessions=0.8×)
- Topic authority via title matching
- Rejected Small-to-Big retrieval (chunks already 1847 chars)

**Results**: 0% → 90% accuracy in 7.5 hours, exceeded 60-70% target by 20-30pp

---

### ADR-010: Phase 9 Agentic RAG Architecture
**File**: [decisions/ADR-010-Agentic-RAG-Architecture.md](decisions/ADR-010-Agentic-RAG-Architecture.md)  
**Date**: January 6-16, 2026  
**Status**: ✅ Complete

Transform RAG from static library to intelligent infrastructure co-pilot with real-time system awareness.

**Key Decisions**:
- Function calling (structured tools) with rule-based intent classification
- 5 MCP tools: Prometheus, Nomad, Consul for live infrastructure queries
- Read-only safety enforcement (100% GET operations, zero mutations)
- ADR Threshold Injector: Embed thresholds in tool responses for context-aware diagnostics
- Glass Box AI: 3-tab visualization (Thought Trace, Timeline, Performance)

**Results**: 98.7% accuracy (157/159 tests), 21-121ms response times (41-238× faster than 5000ms target), 100% cross-reference accuracy, 190 automated tests passing

**Innovations**: ADR Threshold Injector (single-step diagnostics), Glass Box AI (transparent reasoning)

---

### ADR-011: Phase 10 Performance & Intelligence Optimization
**File**: [decisions/ADR-011-PORTFOLIO.md](decisions/ADR-011-PORTFOLIO.md)  
**Date**: January 6-24, 2026  
**Status**: ✅ Complete (Grade: A+)

Transform unreliable AI (69% fallback) to production-ready infrastructure co-pilot (18% fallback).

**Key Decisions**:
- Chain of Verification (CoVe): 7-component tiered system with 3ms overhead
- TTL caching with jitter: 605,000× cached speedup
- Multi-factor confidence scoring: 60/30/10 split (verification/source/length)
- ADR Threshold Injector: Single-step diagnostics with architectural context
- Glass Box AI enhancements: Complete transparency into verification process

**Results**: 18% fallback (51pp improvement), 605,000× speedup, 0% metadata dumps, 416 tests passing

**Innovations**: Confidence Paradox diagnostic (51pp from 50 lines), TTL Jitter pattern (cache stampede prevention)

---

### ADR-012: Multi-Hop Reasoning Architecture (Phase 11)
**File**: [decisions/ADR-012-PORTFOLIO.md](decisions/ADR-012-PORTFOLIO.md)  
**Date**: February 5-13, 2026  
**Status**: ✅ Complete (Grade: A)

Transform single-query RAG into intelligent multi-hop reasoning with conservative confidence strategy.

**Key Decisions**:
- Query expansion with WordNet + spaCy (500+ domain terms)
- Pattern-based classification (5 patterns: comparison, troubleshooting, integration, process, capability)
- Conservative confidence strategy (stop on first LOW confidence sub-query)
- SOLID architecture (decomposition vs execution separation)
- Parallel execution (independent sub-queries, 33% time savings)

**Results**: 90% multi-hop success, 100% HIGH confidence, zero regressions across 3,744 test executions

**Innovations**: Conservative Confidence (knowing when to stop), Pattern-Based Classification (100% accuracy, <1ms)

---

### ADR-013: LLM-Guided Query Decomposition (Phase 13+14)
**File**: [decisions/ADR-013-PORTFOLIO.md](decisions/ADR-013-PORTFOLIO.md)  
**Date**: January-February 2026  
**Status**: ✅ Complete

Hybrid LLM-guided decomposition with semantic understanding and deterministic validation.

**Problem**: Pattern-based decomposition hit 15% GOOD rate ceiling, generating generic sub-questions

**Solution**:
- LLM intent classification (7 types: temporal, aggregation, conditional, comparison, process_flow, integration_chain, diagnostic)
- Entity-aware sub-question generation (113-term vocabulary)
- Conservative threshold (0.5 confidence minimum)
- Three-layer validation (entity coverage, duplication detection, quality assessment)

**Results**:
- 4.4× improvement (15% → 66.7% GOOD rate)
- 2× precision improvement (50% → 100%)
- 7 operational intent types
- 3.95s latency (21% under 5s target)

**Why It Matters**: Hybrid architectures (LLM understanding + pattern structure) beat either approach alone

---

### ADR-014: Conversational Memory & Latency Optimization (Phase 15)
**File**: [decisions/ADR-014-PORTFOLIO.md](decisions/ADR-014-PORTFOLIO.md)
**Date**: February 3-23, 2026
**Status**: ✅ Complete (Grade: A+ / A+ / A+)

Transform RAG from query-response to conversational intelligence with temporal context awareness.

**Key Decisions**:
- Dual-track classification: LLM primary + pattern fallback (100% recall)
- Prompt caching: 32.4% latency reduction via strategic cache boundaries
- Simple prepend coreference: 100% accuracy, <10ms overhead
- SQLite session lifecycle: Conversation memory with intent-aware scoping
- RAGAS validation: External framework for quality assurance

**Results**: 100% recall (+15pp), 4.22s P95 (-33.5%), 100% coreference, 91.7% multi-turn success, 1.000 RAGAS Context Precision

**Innovations**: Cache utilization > token efficiency principle, simple prepend strategy (beating complex ML), professional rollback acceptance

**External Validation**: "Top 1% of local RAG implementations" (Gemini), methodology recognition: "Benchmark → Experiment → Validate → Pivot"

---

### ADR-015: Analytical Intelligence & Operational Guardrails (Phase 16)
**File**: [decisions/ADR-015-PORTFOLIO.md](decisions/ADR-015-PORTFOLIO.md)
**Date**: February 11-25, 2026
**Status**: ✅ Complete (Grade: A+)

Transform high-performance RAG into self-aware analytical system with proactive monitoring.

**Key Decisions**:
- Conversational memory: Limit=3 with cache optimization (7.05ms overhead)
- Metric snapshots: EAV schema with FIFO retention (5.66ms INSERT)
- Hybrid anomaly detection: Statistical bounds + pattern matching (100% accuracy)
- Adaptive sliding window: Cache-stable architecture (<1ms overhead, 7.4% token usage)
- Entity Registry: Temporal coreference with entity pinning (100% pronoun resolution)

**Results**: Self-aware system, 100% anomaly detection, 100% coreference accuracy, 552 tests (zero regressions), sustained A+ grade

**Innovations**: Cache-stable sliding window, hybrid anomaly detection (zero false positives), temporal entity tracking (prevents hallucination)

**External Validation**: "Self-aware analytical system", "Top 1% of local RAG implementations" (Gemini A+)

---

### Phases 17–19: Zero-LLM Analytical Streak

These phases extended the zero-LLM analytical constraint without standalone ADRs. Key decisions are documented in their journey narratives.

| Phase | Focus | Key Achievement |
|-------|-------|-----------------|
| 17 | Advanced Synthesis | 7-category conflict detection, 0.208ms overhead, Glass Box 6 tabs |
| 18 | Predictive Diagnostics | OLS regression, ~0.05ms P95, observer effect, Glass Box Tab 7 |
| 19 | Cross-Modal Correlation | Nomad/Consul event correlation, <10ms Top Cause, 4-state UI |

---

### ADR-016: Diagnostic Context Architecture — Bounded State for Conversational AIOps (Phase 20)
**File**: [decisions/ADR-016-PORTFOLIO.md](decisions/ADR-016-PORTFOLIO.md)
**Date**: 2026-04-06
**Status**: ✅ Accepted (Grade: A+)

Closes the Stateless Oracle problem: the system now remembers its diagnostic conclusions across conversation turns and verifies them against live infrastructure state.

**Key Decisions**:
- `DiagnosticContext` dataclass (7 fields): structured causal attribution, not raw event data
- Context window guard: single bounded function is the **only** injection path (≤200 chars)
- Dual-layer truncation (field-level + string-level): invariant survives future template changes
- TTL=300s (Human Deliberation Buffer), env-var configurable (Observability Tempo)
- Pure function return vs side-effect: testable without Streamlit mocking
- New-query override: metric + spike signals always win over follow-up phrase match
- Verifiable Inference: live trend appended on follow-up, graceful degradation on failure
- Diagnostic Thread (Tab 8): append-only session investigation timeline, Pivot Event natural

**Results**: 12 tests, 999 total tests (980/980 runnable @ 100%), A+ (Gemini CP1 + CP2 + CP3)

**Gemini Concepts Coined**: Verifiable Inference, Diagnostic Thread, Pivot Event, Human Deliberation Buffer, Observability Tempo

> "The completion of Phase 20 marks a definitive shift from building a 'tool' to engineering a 'system.'"

---

### ADR-018: PostgreSQL Migration — Distributed Data Plane (Phase 21)
**File**: [decisions/ADR-018-PORTFOLIO.md](decisions/ADR-018-PORTFOLIO.md)
**Date**: April 2026
**Status**: ✅ Complete (Grade: A+)

Break the single-node pin: migrate the observability database from a SQLite file on the development workstation to PostgreSQL on a dedicated cluster node, via a backend-agnostic connection factory.

**Key Decisions**:
- 137-line connection factory routes by URL scheme (`postgresql://` → Psycopg 3, `sqlite:///` → internal wrapper, unknown → `ValueError`)
- Transparent dialect translation: consumer code writes Psycopg-native `%s`; an internal wrapper translates to SQLite's `?` on every `execute()`
- Python-side timestamp cutoffs: compute `int(time.time()) - seconds` in application code, pass as bound parameter — eliminates the largest class of dialect branching
- `is_sqlite_connection()` predicate as minimum-surface dialect gate for the rare cases where SQL expressions genuinely differ
- Big-bang cutover via single env-var flip — no dual-write, no phased migration

**Results**: Compute decoupled from a single node; single SQL dialect in consumer code; backend-agnostic retention; rollback is one env-var flip

**Innovations**: Python-side cutoff pattern (eliminates SQL dialect divergence on temporal queries); explicit-error routing (no silent backend fallbacks)

---

### ADR-019: Alerting Integration — Webhook Enrichment Pattern (Phase 21)
**File**: [decisions/ADR-019-PORTFOLIO.md](decisions/ADR-019-PORTFOLIO.md)
**Date**: April 2026
**Status**: ✅ Complete (Grade: A+)

Surface Top Cause reasoning to operators at the point of alert, transforming the system from a passive observer into an active participant in infrastructure health — without coupling to an HTTP server in this phase.

**Key Decisions**:
- Implement enrichment as a pure function: `enrich_alert(payload, db_url) → enriched_payload`
- Graceful degradation is first-class: every failure mode returns the alert unchanged (six documented modes, bare exception handler at function boundary)
- Defer HTTP wrapper: logic + tests now; transport layer when load-bearing
- Stable annotation format contract (`correlation_candidate:` prefix, structured `{type=, score=, inflection=}`) — treated as public API
- Zero-LLM analytical stack maintained on the alert path (regex extraction + scored correlation lookup + string formatting)

**Results**: Top Cause reasoning reaches the operator at the point of need; six failure modes documented and tested; 24 tests in four groups (extraction, node parse, graceful degradation, format contract)

**Innovations**: "Fail open" posture for alert enrichment (a system between a critical signal and its recipient should default to pass-through, never interception)

---

### ADR-020: Nomad-Native Discovery + Job-File Placement Policy (Phase 21)
**File**: [decisions/ADR-020-PORTFOLIO.md](decisions/ADR-020-PORTFOLIO.md)
**Date**: April 2026
**Status**: ✅ Complete (Grade: A+)

Heterogeneous cluster placement policy: encode "should this job ever run on this node?" as architectural intent in version-controlled job files, not as ephemeral operational state.

**Key Decisions**:
- Two mechanisms, two purposes: node eligibility flag = operational state (cluster-wide, ephemeral); job-file constraint = architectural intent (job-specific, durable)
- Belt-and-suspenders: use both — constraint as safety net, eligibility flag as scheduling optimization
- Remove `service {}` blocks from jobs pinned to nodes without a Consul agent; rely on Nomad-native service discovery + task-level `check {}` blocks
- Standing convention: any future Linux Docker job carries the macOS-exclusion constraint in its job file

**Results**: Immediate unblocking of 4 services after a forced rescheduling; durable git-traceable placement policy; Linux containers cannot accidentally schedule on the macOS workstation regardless of operational state changes

**Innovations**: Operational-state vs architectural-intent insight (placement policy as ephemeral state is a time bomb); forced rescheduling as an unintentional cluster-wide constraint validation pass

---

### Phase 22: Self-Healing Foundations

Phase 22 introduced Level 0 enrichment and Level 1 dry-run execution without standalone ADRs — the architectural decisions (three-gate safety, kill-switch / rate-limit / loop-guard semantics, Streamlit AppTest harness) are documented in the journey narrative. The phase established the safety architecture that ADR-025 (Phase 23) builds on.

| Phase | Focus | Key Achievement |
|-------|-------|-----------------|
| 22 | Self-Healing Foundations | Level 1 dry-run executor with three-gate safety, 8/8 Glass Box tabs under harness, 1108 tests (0 failing), zero-LLM streak 7 |

---

### ADR-025: 5-Gate Live Execution Pipeline — Deterministic Autonomous Remediation (Phase 23)
**File**: [decisions/ADR-025-PORTFOLIO.md](decisions/ADR-025-PORTFOLIO.md)
**Date**: May 2026
**Status**: ✅ Complete (Grade: A+)

Bridge from advisory (Level 1) to autonomous remediation (Level 2) without an LLM in the safety-critical execution path. Implement as a deterministic state machine with five independently-testable gates.

**Key Decisions**:
- Five gates in series: Pre-flight (eleven failure event types in 60s lookback) → Action → Verify → Stable-duration (60s post-action poll) → Record
- Single-strike rollback: any failure during the 60s window triggers revert (no "two strikes" confirmation pattern)
- Three independent safety controls: pattern-level `live_enabled` flag, global kill-switch (ADR-026), stable-duration rollback — each can abort without coordination
- Two action classes shipped (restart-class + script-class) to prove pipeline generalizes beyond a single remediation type
- ExternalID + CreateTime allocation tracking with 2-second drift tolerance (rejects unrelated allocations from rollback decisions)
- Nanosecond-precision audit trail (causal analysis after incidents requires ordering events within the same millisecond)

**Results**: 5 gates, 2 action classes, 3 safety controls, 11 pre-flight failure types, 0 LLM calls in execution path, 8 consecutive zero-LLM phases (16–23), 27 new tests

**Innovations**: Cognition/execution separation (the architectural payoff that admits LLM-as-planner in the next phase without disturbing the safety contract); single-strike rollback asymmetry (err on the side of reverting)

**External Validation**: "We built a deterministic, safe, and verifiable autonomous remediation system that operates with nanosecond precision and human-in-the-loop overrides" (Gemini CP2)

---

### ADR-026: Kill-Switch with Confirmation Gate — Functional Safety Over Widget Elegance (Phase 23)
**File**: [decisions/ADR-026-PORTFOLIO.md](decisions/ADR-026-PORTFOLIO.md)
**Date**: May 2026
**Status**: ✅ Complete (Grade: A+)

Global emergency brake for live execution: operator-controlled, persisted across restarts, surfaced in CLI and UI with state shared via a JSON state file.

**Key Decisions**:
- Asymmetric confirmation: Enable is single-click (frictionless safer direction); Disable requires a confirmation modal (gated unsafe direction) — mirrors emergency-stop design in industrial equipment
- UI panel rendered at top of `main()`, never inside conditional render blocks — guarantees the safety control is visible at every moment of the operator's session
- Button pair (not `st.toggle`): explicit click-to-action mapping, no carry-forward state semantics, cleaner under AppTest, naturally admits the confirmation gate
- CLI + UI share the same JSON state file (toggle either surface, the other reflects on next render)

**Results**: 2 surfaces, 3 design iterations to land, 3 validation layers required to surface the placement bug, 1 multi-step lifecycle AppTest locked in

**Innovations**: "Functional safety over widget elegance" (the visually elegant choice was wrong for a control that mediates infrastructure mutations); always-visible placement principle ("a safety control that is sometimes visible is, for the moments it is not visible, equivalent to one that does not exist")

**External Validation**: Misdiagnosis-to-resolution arc characterized as "the mark of mature engineering" — bug shipped twice, but the methodology innovation it produced (ADR-027) ensures it cannot ship a third time

---

### ADR-027: Three-Layer UI Validation — A Testing Standard Encoded From a Real Bug (Phase 23)
**File**: [decisions/ADR-027-PORTFOLIO.md](decisions/ADR-027-PORTFOLIO.md)
**Date**: May 2026
**Status**: ✅ Complete (Grade: A+)

Codify three-layer validation as a Testing-Guide standard: every UI feature must pass function-level tests, AppTest integration, AND manual smoke test before it can be declared complete.

**Key Decisions**:
- Layer 1 (function-level): pure logic, return-value contracts, edge-case input handling
- Layer 2 (AppTest): single-turn harness wiring, state persistence, callback registration
- Layer 3 (manual smoke): multi-turn rendering, conditional-render gaps, ergonomics, "does this actually feel right"
- Layer 3 is mandatory, not optional: optional manual validation degrades to never-performed manual validation under deadline pressure
- Wired into the Session 9 completion gate: a feature without Layer 3 evidence is not eligible for the session completion summary
- ~3-5 minutes per feature — judged worth the cost for the bug class it catches

**Results**: Codified into Testing-Guide.md v5.4; integrated into CHAT-SESSION-WORKFLOW completion gate; the kill-switch placement bug (which passed Layers 1 and 2 but failed Layer 3) becomes the canonical worked example

**Innovations**: "Lesson encoded as a process" — the same pattern Phase 17 used for the UI Smoke Test Gate, Phase 21 used for the Sanitization Completeness Checklist, and the multi-phase workflow used for the Session Summary Gate. Single-incident learning converted into a permanent process control

**External Validation**: "Three-Layer UI Validation is a portfolio-grade pattern that generalizes well beyond this specific incident" (Gemini CP2)

---

## Story Arc: Foundation → Build → Optimize → Intelligence → Observability Trilogy → Self-Healing → Autonomous Bridge

**ADR-001** (Foundation): Choose simple, reliable orchestration  
**ADR-008** (Build): Create production RAG platform  
**ADR-009** (Optimize): Achieve 90% accuracy through systematic engineering  
**ADR-010** (Intelligence): Transform into agentic co-pilot with live system awareness  
**ADR-011** (Optimize): 18% fallback, 605,000× speedup, CoVe  
**ADR-012** (Intelligence Enhancement): Multi-hop reasoning with conservative confidence  
**ADR-013** (Intelligence Enhancement): LLM-guided query decomposition with hybrid architecture  
**ADR-014** (Intelligence + Performance): Conversational memory with elite-tier optimization ("top 1%")  
**ADR-015** (Self-Awareness): Analytical intelligence with proactive monitoring and temporal coreference  
**Phases 17–19** (Observability): Synthesis → Prediction → Explanation (zero-LLM streak 3 → 4)  
**ADR-016** (Conversational AIOps): Memory + verification — predict, explain, remember, verify  
**ADR-018/019/020** (Distributed Foundations): PostgreSQL migration, alerting integration, placement policy — first containerized cross-node services  
**Phase 22** (Self-Healing): Three-gate safety architecture with Level 1 dry-run executor, Streamlit AppTest harness for 8/8 Glass Box tabs  
**ADR-025/026/027** (Autonomous Bridge): 5-gate live execution pipeline, kill-switch confirmation gate, Three-Layer UI Validation standard ⭐

---

## Architecture Diagrams

Visual representations of system architecture.

**Available Now**:
- [Infrastructure Architecture (Phase 6)](diagrams/infrastructure-architecture-phase6.png) - 9-layer stack before RAG (foundation)
- [Infrastructure Architecture (Phase 7)](diagrams/nomad_cluster_architecture-phase7.png) - Complete 10-layer stack with RAG platform
- [RAG Pipeline Architecture (Phase 8)](diagrams/rag-pipeline-architecture-phase8.png) - 5-layer pipeline with three-layer boosting system
- [Phase 9 System Topology](diagrams/phase-9-system-topology.png) - MCP tools integration with infrastructure (5-layer agentic architecture)
- [Phase 9 Data Flow](diagrams/phase-9-data-flow.png) - Agentic reasoning pipeline with Glass Box AI (8-step execution)
- [Phase 10 System Topology + CoVe Detail](diagrams/phase-10-system-topology.png) - Performance optimization with 7-component verification flow
- [Phase 21 Distributed System Topology](diagrams/phase-21-system-topology.png) - 4-node cluster with PostgreSQL migration and cross-node data flows
- [Phase 22-23 Observation to Autonomy](diagrams/phase-22-23-observation-to-autonomy.png) - 5-gate live execution pipeline with three independent safety controls ⭐ NEW

**See**: [Diagrams README](diagrams/README.md) for detailed explanations, usage guidance, and technical details

---

## Technology Stack

Complete overview of technologies used across all phases.

**See**: [Complete Technology Stack](tech-stack.md)

**Categories**:
- **Infrastructure & Orchestration**: Nomad, Consul, Traefik, Docker, Proxmox, NFS
- **Observability & Monitoring**: Prometheus, Grafana, AlertManager, Loki, Exporters
- **AI/ML Platform**: Ollama (hybrid CPU+GPU), embedding models, LLMs
- **RAG Platform**: Streamlit, SQLite, hybrid search, document boosting, caching
- **Agentic Systems**: MCP tools (Prometheus, Nomad, Consul), intent classification, cross-reference engine
- **AIOps**: OLS regression engine, correlation engine, DiagnosticContext, Verifiable Inference
- **Development & Process**: Python, pytest, UV, Git, ADR framework
- **Hardware & Network**: 4-node cluster (3 Proxmox VMs + ML workstation), GPU, NAS

**Philosophy**: Operational simplicity prioritized over complexity. Resource efficiency on older hardware. Production-grade patterns with learning value.

**Achievements**: 98.7% agentic accuracy, 90% RAG accuracy, 24× GPU speedup, 605,000× cache speedup, 1179 tests @ 100% (0 failing), zero-LLM streak 8 phases (16-23), 5-gate live execution pipeline with three-gate safety

---

**Status**: Phase 23 complete — The Autonomous Bridge: 5-gate live execution with three independent safety controls ✅  
**Latest**: ADR-025/026/027 (Phase 23, Grade A+) — live execution pipeline, kill-switch confirmation gate, Three-Layer UI Validation standard  
**Zero-LLM streak**: 8 consecutive phases (16–23)
