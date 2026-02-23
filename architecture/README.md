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

**Innovations**: Cache utilization > token efficiency principle, simple prepend strategy (beating complex ML), professional rollback acceptance (quantization)

**External Validation**: "Top 1% of local RAG implementations" (Gemini), methodology recognition: "Benchmark → Experiment → Validate → Pivot"

---

## Story Arc: Foundation → Build → Optimize → Intelligence

**ADR-001** (Foundation): Choose simple, reliable orchestration  
**ADR-008** (Build): Create production RAG platform  
**ADR-009** (Optimize): Achieve 90% accuracy through systematic engineering  
**ADR-010** (Intelligence): Transform into agentic co-pilot with live system awareness  
**ADR-011** (Optimize): 18% fallback, 605,000× speedup, CoVe  
**ADR-012** (Intelligence Enhancement): Multi-hop reasoning with conservative confidence  
**ADR-013** (Intelligence Enhancement): LLM-guided query decomposition with hybrid architecture  
**ADR-014** (Intelligence + Performance): Conversational memory with elite-tier optimization ("top 1%")

---

## Architecture Diagrams

Visual representations of system architecture.

**Available Now**:
- [Infrastructure Architecture (Phase 6)](diagrams/infrastructure-architecture-phase6.png) - 9-layer stack before RAG (foundation)
- [Infrastructure Architecture (Phase 7)](diagrams/nomad_cluster_architecture-phase7.png) - Complete 10-layer stack with RAG platform
- [RAG Pipeline Architecture (Phase 8)](diagrams/rag-pipeline-architecture-phase8.png) - 5-layer pipeline with three-layer boosting system
- [Phase 9 System Topology](diagrams/phase-9-system-topology.png) - MCP tools integration with infrastructure (5-layer agentic architecture)
- [Phase 9 Data Flow](diagrams/phase-9-data-flow.png) - Agentic reasoning pipeline with Glass Box AI (8-step execution)

**See**: [Diagrams README](diagrams/README.md) for detailed explanations, usage guidance, and technical details

**Coverage**: Pre-RAG foundation (9 layers) + Complete infrastructure (10 layers) + RAG optimization (3-layer boosting) + Agentic intelligence (5 MCP tools + Glass Box AI)

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
- **Development & Process**: Python, pytest, UV, Git, ADR framework
- **Hardware & Network**: 3-node cluster, Mac Studio GPU, 10TB NAS

**Philosophy**: Operational simplicity prioritized over complexity. Resource efficiency on older hardware. Production-grade patterns with learning value.

**Achievements**: 98.7% agentic accuracy, 90% RAG accuracy, 24× GPU speedup, 200× cache speedup, 99.9%+ uptime

---

**Status**: Content being added incrementally based on phase completion  
**Latest**: Phase 15 complete - Conversational Memory & Latency Optimization ("Top 1%") ✅
```

---

## Git Commit Message Template
```
Architecture README: Add Phase 15 ADR-014

Added ADR-014 section documenting Phase 15 achievements:
- Conversational memory & latency optimization
- 100% recall, 4.22s P95 (-33.5%), 91.7% multi-turn
- "Top 1%" external validation (Gemini)
- Methodology recognition: "Benchmark → Experiment → Validate → Pivot"

Updated Story Arc with ADR-014 (Intelligence + Performance)
Updated footer: Phase 15 complete

Phase 15 Key Innovations:
- Dual-track classification (100% recall)
- Prompt caching (-32.4% latency)
- Simple prepend coreference (100% accuracy, <10ms)
- SQLite session lifecycle (conversational memory)
- RAGAS validation framework (1.000 Context Precision)
