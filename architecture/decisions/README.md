# Architecture Decision Records

This folder contains the architectural decision records from work on
the homelab platform. Some of the earliest — including ADR-001 on
the orchestration platform choice — still hold today.

## Three ADRs worth reading first

The most technically sophisticated ADRs in this folder are not
necessarily the most informative ones. The three below are chosen
for what they show about method, not outcome.

**[ADR-020: Job placement as architectural intent](ADR-020-PORTFOLIO.md)** —
Distinguishes operational state (Consul de-registration, node eligibility
flags) from architectural intent (job-file placement constraints).
The distinction is the difference between fixing a symptom and encoding
a design decision so it survives the next operational change.

**[ADR-026: Lifting the kill-switch confirmation gate](ADR-026-PORTFOLIO.md)** —
Captures a small operational fix — moving a confirmation gate out of a
conditional block in the chat handler — at full ADR weight. The discipline
of writing things down isn't gated by scope; the reasoning behind a small
fix is recorded the same way as a platform choice.

**[ADR-027: Three-layer UI validation](ADR-027-PORTFOLIO.md)** — Converts
a specific bug (the kill-switch UI placement issue, which shipped twice)
into a process standard requiring function-level, AppTest, and manual-smoke
validation for stateful widget interactions. The move that separates
engineers who fix problems from engineers who eliminate the category
of problem.

---

## Phase 22-23 — From Observation to Autonomy ⭐ NEW

### [ADR-025: Live Execution Pipeline (Phase 23)](ADR-025-PORTFOLIO.md)
**Decision**: 5-gate pipeline for autonomous remediation — kill-switch → rate-limit → pre-flight → restart → stable-duration — with single-strike rollback (any single failure → rolled_back).
**Context**: Phase 23 elevated dry-run capability (Phase 22) to live execution. Crossing the action boundary requires deterministic, auditable safety: any LLM-driven decision in the execution path was excluded by design.
**Consequence**: Live execution shipped without an LLM in the gate-and-restart path. Eight consecutive zero-LLM phases. Validated against the searxng restart in production (HTTP 200, 0.4s end-to-end).

### [ADR-026: Kill-Switch Confirmation Gate (Phase 23)](ADR-026-PORTFOLIO.md)
**Decision**: Two-button Yes/Cancel confirmation gate for the kill-switch toggle, lifted from the `if prompt:` rerun-disappearing block to `main()` after the conversation_id early-return.
**Context**: Phase 23 Session 7 manual smoke test exposed that click-driven Streamlit reruns were destroying the confirmation state. The architectural fix (relocate the panel) was the third iteration after two patch attempts that papered over symptoms.
**Consequence**: Operator-intent kill-switch now survives reruns; the confirmation-gate placement constraint is encoded in the Three-Layer UI Validation standard (ADR-027) so future widget interactions don't repeat the discovery cycle.

### [ADR-027: Three-Layer UI Validation Standard (Phase 23)](ADR-027-PORTFOLIO.md)
**Decision**: Function-level / AppTest / Manual-smoke triad mandatory for any stateful Streamlit widget interaction (codified in Testing-Guide v5.4).
**Context**: The kill-switch panel relocation (ADR-026) was caught only by manual smoke testing — function-level tests passed, AppTest passed, but a real click-driven rerun destroyed the state. Each of the three layers catches a different bug class.
**Consequence**: Standardized validation ladder for stateful UI code; closes the gap exposed in Phase 22 Session 7 and Phase 23 Sessions 4 and 7.

---

## Phase 21 — Distributed Foundations

### [ADR-020: Nomad-Native Discovery + Job-File Placement Policy](ADR-020-PORTFOLIO.md)
**Decision**: Encode placement constraints directly in job files rather than relying on Consul service tags or runtime operator commands.
**Context**: Docker restart on lab-node-3 triggered Consul de-registration, forcing emergency rescheduling and revealing that operational state (Consul health) diverges from architectural intent (which nodes a job should run on).
**Consequence**: Durable placement — job file is the authoritative source of scheduling intent regardless of Consul state.

### [ADR-019: Alerting Integration Architecture](ADR-019-PORTFOLIO.md)
**Decision**: Implement alert enrichment as a pure function (`enrich_alert(payload, db_url) → dict`) without bundling an HTTP server.
**Context**: Phase 20 introduced correlation engine "Top Cause" output in the UI, but Alertmanager notifications remained disconnected from that reasoning — operators saw raw metric firings without the correlation context the UI already had. HTTP server binding to a cluster IP would also create placement constraints and restartability issues, so enrichment as a pure function was preferred over bundling a server.
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

**ADR Count**: 16 portfolio ADRs published (ADR-001, ADR-008–016, ADR-018–020, ADR-025–027)
**Coverage**: Infrastructure → RAG → Agentic → Analytics → PostgreSQL Migration → Self-Healing → Autonomous Execution
**Status**: All published decisions documented with context, options, rationale, and consequences
