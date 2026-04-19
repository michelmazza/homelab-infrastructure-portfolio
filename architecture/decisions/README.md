# Architecture Decision Records

Key architectural decisions documented across the infrastructure and AI/ML engineering journey.

---

## Phase 21 — Distributed Foundations ⭐ NEW

### [ADR-020: Nomad-Native Discovery + Job-File Placement Policy](ADR-020-PORTFOLIO.md)
**Decision**: Encode placement constraints directly in job files rather than relying on Consul service tags or runtime operator commands.
**Context**: Docker restart on lab-node-3 triggered Consul de-registration, forcing emergency rescheduling and revealing that operational state (Consul health) diverges from architectural intent (which nodes a job should run on).
**Consequence**: Durable placement — job file is the authoritative source of scheduling intent regardless of Consul state.

### [ADR-019: Alerting Integration Architecture](ADR-019-PORTFOLIO.md)
**Decision**: Implement alert enrichment as a pure function (`enrich_alert(payload, db_url) → dict`) without bundling an HTTP server.
**Context**: Alertmanager webhook pattern established in ADR-019 as scope-minimal enrichment. HTTP server binding to a cluster IP creates placement constraints and restartability issues.
**Consequence**: Enrichment logic is testable in isolation; HTTP server can be added later without coupling.

### [ADR-018: PostgreSQL Migration Strategy](ADR-018-PORTFOLIO.md)
**Decision**: Introduce `rag/db.py` as a thin connection factory routing `postgresql://` to Psycopg 3 and `sqlite:///` to `_SQLiteWrapper`, with `%s` placeholder translation transparent to callers.
**Context**: SQLite on Mac Studio became a single point of failure and prevented cross-node data access. PostgreSQL on a cluster node enables distributed writes.
**Consequence**: Big-bang cutover avoided — SQLite remains the default for all tests; PostgreSQL activates only when `DB_URL` env var is set. Five observability modules ported with `%s` placeholders.

---

## Phase 16 — Analytical Intelligence

### [ADR-016: Zero-LLM Analytical Architecture](ADR-016-PORTFOLIO.md)
**Decision**: All analytical pipeline components (conflict detection, synthesis, prediction, correlation) must be pattern-based with deterministic, latency-bounded outputs — no LLM calls in the analytical path.
**Consequence**: Zero-LLM streak sustained for 6 consecutive phases (16–21). Every analytical output is explainable by reading the code.

---

## Phase 15 — Robustness & Latency

### [ADR-015: Adaptive Context Window](ADR-015-PORTFOLIO.md)
**Decision**: Dynamic sliding window for conversation context (not fixed size).
**Consequence**: 63% token reduction (20% → 7.4%), P95 latency maintained, cache stability 100%.

### [ADR-014: Conversational Memory Architecture](ADR-014-PORTFOLIO.md)
**Decision**: Prepend-based conversation history (simple, cache-stable).
**Consequence**: Beat ML-based memory approaches on both performance and latency.

---

## Phase 13–14 — LLM-Guided Decomposition

### [ADR-013: Query Decomposition Strategy](ADR-013-PORTFOLIO.md)
**Decision**: LLM-guided intent decomposition with 7 intent types.
**Consequence**: 4.4× GOOD rate improvement (15% → 66.7%), 100% precision.

---

## Phase 12 — Retrieval Optimization

### [ADR-012: Corpus Expansion Strategy](ADR-012-PORTFOLIO.md)
**Decision**: Data-driven corpus expansion over ML-based document processing (Docling).
**Consequence**: 92% hit rate (+20pp), 3.4× corpus growth, 4-6× ROI vs Docling.

---

## Phase 11 — Multi-Hop Reasoning

### [ADR-011: Multi-Hop Retrieval Architecture](ADR-011-PORTFOLIO.md)
**Decision**: Pattern-based classification + parallel execution for multi-hop queries.
**Consequence**: 90% routing success, 100% HIGH confidence, 33% time savings.

---

## Phase 9–10 — Agentic RAG + Performance

### [ADR-010: Agentic RAG Architecture](ADR-010-Agentic-RAG-Architecture.md)
**Decision**: MCP tool pattern with ADR Threshold Injector for live infrastructure integration.
**Consequence**: 98.7% agentic accuracy, Glass Box AI (now 8 tabs through Phase 20).

### [ADR-009: Retrieval Optimization](ADR-009-retrieval-optimization.md)
**Decision**: Three-layer boosting (Hybrid Search + Document Type + Topic Authority).
**Consequence**: 90% accuracy (Phase 8 baseline established for all future phases).

### [ADR-008: Advanced RAG Platform](ADR-008-advanced-rag-platform.md)
**Decision**: Production RAG with semantic caching and hybrid BM25 + vector search.
**Consequence**: Foundation for all Phase 9+ agentic capabilities.

---

## Foundation — Phase 1-3

### [ADR-001: Orchestration Platform](ADR-001-orchestration-platform.md)
**Decision**: HashiCorp Nomad over Kubernetes for homelab orchestration.
**Consequence**: Operational simplicity, Consul integration, foundation for all subsequent phases.

---

**ADR Count**: 20 (ADR-001 through ADR-020)
**Coverage**: Infrastructure → RAG → Agentic → Analytics → PostgreSQL Migration
**Status**: All decisions documented with context, options, rationale, and consequences
