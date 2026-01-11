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

## Story Arc: Foundation → Build → Optimize

**ADR-001** (Foundation): Choose simple, reliable orchestration  
**ADR-008** (Build): Create production RAG platform  
**ADR-009** (Optimize): Achieve 90% accuracy through systematic engineering

---

## Architecture Diagrams

Visual representations of system architecture.

**Available Now**:
- [Infrastructure Architecture (Phase 7)](diagrams/infrastructure-architecture-phase7.png) - Complete 10-layer stack showing Nomad cluster, Consul service mesh, Traefik routing, observability, AI/ML platform, and RAG knowledge system
- [RAG Pipeline Architecture (Phase 8)](diagrams/rag-pipeline-architecture-phase8.png) - 5-layer pipeline with three-layer boosting system (Hybrid Search + Document Type + Topic Authority)

**See**: [Diagrams README](diagrams/README.md) for detailed explanations, usage guidance, and technical details

**Coverage**: Complete infrastructure topology (10 layers) + RAG optimization architecture (3-layer boosting)

---

## Technology Stack

Complete overview of technologies used across all phases.

**See**: [Complete Technology Stack](tech-stack.md)

**Categories**:
- **Infrastructure & Orchestration**: Nomad, Consul, Traefik, Docker, Proxmox, NFS
- **Observability & Monitoring**: Prometheus, Grafana, AlertManager, Loki, Exporters
- **AI/ML Platform**: Ollama (hybrid CPU+GPU), embedding models, LLMs
- **RAG Platform**: Streamlit, SQLite, hybrid search, document boosting, caching
- **Development & Process**: Python, Git, pytest, ADR framework
- **Hardware & Network**: 3-node cluster, Mac Studio GPU, 10TB NAS

**Philosophy**: Operational simplicity prioritized over complexity. Resource efficiency on older hardware. Production-grade patterns with learning value.

**Achievements**: 90% RAG accuracy, 24× GPU speedup, 200× cache speedup, 99.9%+ uptime

---

**Status**: Content being added incrementally based on phase completion
