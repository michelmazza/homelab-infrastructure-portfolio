# Technology Stack

Complete overview of technologies used across all phases of the infrastructure evolution.

---

## Overview

This technology stack represents a production-grade homelab built on operational simplicity, resource efficiency, and hands-on learning. Every technology choice was made deliberately, documented in ADRs, and validated through real-world usage.

**Philosophy**: Simple beats complex. Efficient beats resource-heavy. Production-grade patterns beat hobby-grade shortcuts.

---

## Infrastructure & Orchestration

| Technology | Version | Purpose | Phase | Status |
|------------|---------|---------|-------|--------|
| **HashiCorp Nomad** | 1.9.4 | Container orchestration | 2 | ✅ 3-node cluster |
| **HashiCorp Consul** | Latest | Service discovery + mesh | 3 | ✅ 3-node cluster |
| **Traefik** | 2.10 | Reverse proxy + routing | 3 | ✅ Production |
| **Docker** | Latest | Container runtime | 2 | ✅ All nodes |
| **Proxmox VE** | 8.x | Virtualization platform | 1 | ✅ 3 hosts |
| **NFS** | - | Distributed storage | 4 | ✅ 10TB Synology |

**Key Decisions**:
- **Nomad over Kubernetes**: Operational simplicity prioritized (ADR-001). ~200MB overhead vs ~3GB for K3s.
- **Consul for service mesh**: Dynamic service discovery with health checks, zero manual configuration
- **Traefik with Consul integration**: Automatic routing based on Consul catalog, no manual route config
- **NFS for storage**: Simple, reliable, adequate performance for homelab scale

---

## Observability & Monitoring

| Technology | Version | Purpose | Phase | Status |
|------------|---------|---------|-------|--------|
| **Prometheus** | 2.54.1 | Metrics collection | 5 | ✅ 14+ targets |
| **Grafana** | 10.2.2 | Metrics visualization | 5 | ✅ 6 dashboards |
| **AlertManager** | Latest | Alert routing | 5 | ✅ Pushover integration |
| **Loki** | Latest | Log aggregation | 5 | ✅ Operational |
| **Node Exporter** | Latest | Host metrics | 5 | ✅ 3 hosts |
| **Blackbox Exporter** | Latest | Endpoint probes | 5 | ✅ HTTP health checks |
| **Pushover** | - | Mobile notifications | 5 | ✅ Real-time alerts |

**Monitoring Achievements**:
- 14+ Prometheus targets scraped every 15 seconds
- 6 operational Grafana dashboards (Host Overview, Ollama Monitoring, Service Health, etc.)
- 5 alert rules with mobile push notifications (<2 minute delivery)
- Zero false positives in production alerting

---

## AI/ML Platform

| Technology | Version | Purpose | Phase | Status |
|------------|---------|---------|-------|--------|
| **Ollama** | 0.3.14 | LLM inference engine | 6 | ✅ Hybrid (CPU+GPU) |
| **nomic-embed-text** | 768d | Embedding model | 7 | ✅ Production |
| **llama3.1:8b** | 8B params | Chat model | 6-8 | ✅ Primary model |
| **Sentence Transformers** | Latest | Embedding library | 7 | ✅ Production |
| **rank-bm25** | Latest | Keyword search | 8 | ✅ Hybrid search |

**Hybrid Ollama Architecture**:
- **CPU Deployment** (Nomad on lab02): Learning, experimentation, ~5 tokens/sec
- **GPU Deployment** (Mac Studio M1 Max): Production, performance, ~127 tokens/sec (24× faster!)
- **Why both**: Learn on CPU, run production on GPU, educational + performance benefits

---

## RAG Platform (Phases 7-8)

| Technology | Purpose | Implementation | Status |
|------------|---------|----------------|--------|
| **Streamlit** | Chat web UI | Python framework, port 8501 | ✅ Production |
| **SQLite (WAL mode)** | Conversation storage | 132KB database, 100% concurrency | ✅ Production |
| **JSON Embeddings** | Vector storage | 15MB file, 611 chunks, 85 docs | ✅ Production |
| **Semantic Caching** | Query cache | Similarity 0.95, 200× speedup | ✅ Production |
| **Hybrid Search** | BM25 + Vector | α=0.3 keyword/semantic ratio | ✅ 90% accuracy |
| **Document Boosting** | Authority ranking | ADR=1.5×, Reports=1.3×, Sessions=0.8× | ✅ Production |
| **Topic Authority** | Title matching | 1.5× boost for precise queries | ✅ Production |

**RAG Evolution**:
- **Phase 7**: Production platform (85 docs, 100% baseline, 2.5s latency)
- **Phase 8**: Optimization (0% → 90% accuracy, ~25ms search latency)

**Technology Decisions (ADR-008)**:
- **JSON over Vector DB**: Simple, portable, adequate for 85 docs, can migrate if >500 docs
- **SQLite WAL over PostgreSQL**: Single-user, 364.8 ops/sec sufficient, simple deployment
- **Streamlit over Flask**: Rapid UI development, built-in state management

---

## Development & Process

| Technology | Purpose | Usage | Status |
|------------|---------|-------|--------|
| **Python** | Primary language | Scripts, RAG, automation | ✅ Active |
| **Git** | Version control | All configs, code, docs | ✅ Active |
| **uv** | Python env manager | RAG environment, fast | ✅ Active |
| **pytest** | Testing framework | RAG test suite (20 queries) | ✅ Active |
| **Markdown** | Documentation | ADRs, notes, guides | ✅ Extensive |
| **YAML/HCL** | Configuration | Nomad jobs, Prometheus | ✅ IaC |

**Development Practices**:
- Session-based development (60-90 min timeboxed work)
- ADR framework for decisions
- Git discipline (frequent commits, descriptive messages)
- Test-driven optimization (validation before implementation)

---

## AI-Assisted Engineering

| Tool | Purpose | Usage Pattern | Phase |
|------|---------|---------------|-------|
| **Claude (Anthropic)** | Implementation, code generation | Real-time problem solving, pattern research | All |
| **Gemini (Google)** | Architecture review, validation | Strategic feedback, bug catching | All |

**Integration**:
- **Session 0**: AI validates approach before coding
- **Implementation**: Pattern research, code examples
- **Review**: Cross-validation (Claude + Gemini)
- **Learning**: Accelerate understanding of unfamiliar tech

**Phase 8 Example**: Session 0 validated three-pillar strategy with Claude, Session 3 Gemini caught test suite bugs

---

## Hardware

| Component | Specs | Role | Status |
|-----------|-------|------|--------|
| **Proxmox lab01** | Intel i7-2600 (8 cores), 16GB RAM | Server + Client | ✅ Active |
| **Proxmox lab02** | Intel i5-4570 (4 cores), 32GB RAM | Server + Client | ✅ Active |
| **Proxmox lab03** | Intel i5-4590 (4 cores), 32GB RAM | Server + Client (Leader) | ✅ Active |
| **Mac Studio** | M1 Max (10 cores), 32GB RAM | GPU Inference | ✅ Active |
| **Synology NAS** | 10TB storage | NFS Storage | ✅ Active |

**Resource Efficiency**:
- Nomad: ~200MB overhead per node (vs K3s ~3GB)
- Consul: ~50MB per node
- Total cluster overhead: <1GB across all nodes
- Older hardware (2011-2013 CPUs) runs production workloads effectively

---

## Network

| Component | Address/Range | Purpose | Status |
|-----------|---------------|---------|--------|
| **Network** | Private subnet | Homelab subnet | ✅ Active |
| **DNS** | Synology DNS | Internal resolution | ✅ Active |
| **Domain** | mshome.net | Internal domain | ✅ Active |

**Network Architecture**:
- Single subnet for simplicity
- Synology DNS resolves all *.mshome.net to Traefik IP
- Traefik routes based on hostname (Consul integration)
- No manual DNS updates needed (service discovery)

---

## Technology Selection Philosophy

### **Decision Criteria** (from ADR-001)

**Operational Simplicity** (40% weight):
- Can I troubleshoot this at 2am?
- How much overhead does this add?
- What breaks if I need to restart?

**Resource Efficiency** (25% weight):
- Memory footprint on older hardware
- CPU overhead during normal operation
- Storage requirements

**Learning Value** (20% weight):
- Transferable to enterprise environments
- Modern patterns worth knowing
- Skills that advance career

**Ecosystem** (15% weight):
- Community support and documentation
- Integration with other tools
- Long-term viability

### **Key Trade-offs Accepted**

**Nomad over Kubernetes**:
- ✅ Gained: Simplicity, low overhead, easier operations
- ❌ Lost: Kubernetes ecosystem, broader job market

**JSON over Vector DB**:
- ✅ Gained: Simplicity, portability, zero dependencies
- ❌ Lost: Advanced vector search features, scalability >500 docs

**SQLite over PostgreSQL**:
- ✅ Gained: Simple deployment, adequate performance
- ❌ Lost: Multi-user support, replication

**Rationale**: For homelab scale (single operator, <100 docs, <20 services), simplicity beats features. All decisions are reversible if requirements change.

---

## Evolution by Phase

### **Phases 1-3: Foundation**
Proxmox virtualization → Nomad orchestration → Consul service discovery → Traefik routing

### **Phases 4-5: Production Operations**
15+ services deployed → NFS storage integrated → Prometheus/Grafana observability → Alerting with Pushover

### **Phase 6: AI/ML Foundation**
Ollama deployed (CPU) → Hybrid architecture (CPU + GPU) → 24× performance improvement → 11 models available

### **Phase 7: RAG Platform**
Production RAG system → 85 documents embedded → 100% accuracy baseline → Streamlit UI → Conversation persistence

### **Phase 8: Systematic Optimization**
0% → 90% accuracy → Hybrid search (BM25 + Vector) → Document boosting breakthrough (+55pp) → Topic authority (+15pp)

### **Phase 9: Agentic Capabilities** (In Progress)
MCP tools integration → Infrastructure automation → Diagnostic capabilities

---

## Metrics & Achievements

### **Infrastructure**
- ✅ 3-node Nomad cluster, 99.9%+ uptime
- ✅ 15+ containerized services orchestrated
- ✅ 10TB distributed storage via NFS
- ✅ Zero manual service routing (Consul integration)

### **Observability**
- ✅ 14+ Prometheus targets monitored
- ✅ 6 operational Grafana dashboards
- ✅ 5 alert rules with mobile notifications
- ✅ <2 minute alert delivery time
- ✅ Zero false positives in production

### **AI/ML**
- ✅ 24× GPU speedup (Mac Studio vs CPU)
- ✅ 90% RAG accuracy (Phase 8 optimization)
- ✅ 200× cache speedup (semantic caching)
- ✅ ~25ms search latency (production-grade)
- ✅ 11 AI models available (llama3.1, phi3, etc.)

### **RAG Platform**
- ✅ 85 documents embedded (611 chunks)
- ✅ 90% overall accuracy (80% factual, 100% architectural)
- ✅ 40% cache hit rate
- ✅ Conversation persistence (SQLite)
- ✅ Full observability (10 RAG-specific metrics)

---

## Technology Lessons Learned

### **What Worked Well**

**Nomad**: Exceeded expectations. Simple, reliable, low overhead. Right choice for homelab scale.

**Hybrid Ollama**: Best of both worlds. Learn on CPU, run production on GPU. 24× speedup is transformative.

**JSON Embeddings**: "Good enough" solution that's simple and portable. Can migrate to vector DB if needed (>500 docs).

**SQLite WAL Mode**: Surprising performance (364.8 ops/sec). Perfect for single-user RAG application.

**Streamlit**: Rapid UI development. Built-in state management saved weeks of work.

**Hybrid Search (BM25 + Vector)**: Complementary strengths. Keywords catch exact matches, vectors handle semantic queries.

### **What We'd Do Differently**

**Test Suite Earlier**: Should have built test suite in Phase 7, not Phase 8. Delayed optimization insights.

**Document Boosting from Start**: Obvious in retrospect. ADRs are authoritative, should rank higher by default.

**Prometheus Config**: Should have used file-based service discovery from start, not static configs.

### **Future Improvements**

**Vault Integration**: Secrets management is manual. Vault would automate credential rotation.

**Service Mesh (Consul Connect)**: mTLS between services would add security layer.

**Vector DB Migration**: If document count >500, migrate from JSON to Qdrant or Weaviate.

**Multi-GPU Support**: If adding more workloads, could run multiple Ollama instances with GPU sharing.

---

## Technology Recommendations

### **For Similar Homelab Projects**

**Start Here**:
1. Proxmox (virtualization)
2. Nomad (orchestration - simpler than K8s)
3. Consul (service discovery)
4. Traefik (reverse proxy)
5. Prometheus + Grafana (observability)

**Then Add**:
6. Ollama (local LLM experimentation)
7. Streamlit (rapid UI prototyping)
8. SQLite (simple data persistence)

**Avoid Early**:
- Kubernetes (too complex for homelab scale)
- Vector databases (JSON embeddings work <500 docs)
- Multi-node databases (SQLite sufficient for single operator)

### **For Production/Enterprise**

**Different Trade-offs**:
- Kubernetes over Nomad (ecosystem, team knowledge)
- Vector DB over JSON (scale, features)
- PostgreSQL over SQLite (multi-user, replication)
- Managed services over self-hosted (operational burden)

**Same Principles**:
- Validation-first approach (test suites drive optimization)
- Data-driven decisions (measure before changing)
- ADR framework (document decisions)
- Systematic debugging (diagnostic tools before guessing)

---

## Technology Support & Community

**Primary Resources**:
- **HashiCorp Learn**: Nomad/Consul tutorials and guides
- **Ollama Discord**: Active community for LLM deployment
- **Streamlit Forum**: UI component examples and patterns
- **Prometheus/Grafana Docs**: Excellent official documentation

**AI Assistants**:
- **Claude**: Implementation patterns, code examples, real-time debugging
- **Gemini**: Architecture validation, alternative perspectives, bug catching

---

**Status**: Complete technology stack (Phases 1-8)  
**Philosophy**: Operational simplicity, resource efficiency, production-grade patterns  
**Next**: Phase 9 - MCP tools integration for agentic capabilities
