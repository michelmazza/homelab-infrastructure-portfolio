# Architecture Diagrams

Visual representations of system architecture across phases.

---

## Infrastructure Architecture (Phase 7)

**File**: `infrastructure-architecture-phase7.png`  
**Created**: January 4, 2026  
**Updated**: January 11, 2026 (sanitized for portfolio)  
**Source**: draw.io

**Shows Complete 10-Layer Stack**:
- **DNS Layer**: Synology DNS (all services point to single IP)
- **Reverse Proxy Layer**: Traefik v2.10 with Consul catalog provider, host-based routing, Prometheus metrics
- **Service Discovery Layer**: Consul 3-node cluster (service registration, health monitoring, DNS interface)
- **Orchestration Layer**: Nomad 3-node cluster (container orchestration, job scheduling, resource allocation)
- **Application Services**: 15+ containerized services (web apps, search, infrastructure tools)
- **AI/ML Inference Layer**: Hybrid Ollama deployment (CPU cluster-based + GPU Mac Studio)
- **RAG Knowledge Platform**: Streamlit UI, caching, analytics, conversation persistence (Phase 7)
- **Observability Layer**: Prometheus (14+ targets), Grafana (6 dashboards), AlertManager (Pushover integration), exporters
- **External GPU Infrastructure**: Mac Studio M1 Max (24× faster than CPU, 11 AI models)
- **Storage Layer**: Synology NAS 10TB via NFS (persistent data for PostgreSQL, n8n, Gitea, Prometheus, Grafana, Ollama, RAG embeddings)

**Network Details**:
- Cluster subnet: Generic private IPs (sanitized)
- Node configuration: 3-node cluster with leader election
- Service discovery: Automatic via Consul
- Health checks: Comprehensive monitoring at every layer

**Key Architecture Decisions**:
- **Nomad over Kubernetes**: Operational simplicity (ADR-001)
- **Consul service mesh**: Dynamic service discovery
- **Traefik**: Automatic routing with Consul integration
- **Hybrid Ollama**: CPU for learning, GPU for performance
- **NFS storage**: Simple, reliable, adequate performance

**Use For**: 
- Infrastructure discussions and system design interviews
- Explaining multi-layer architecture approach
- Demonstrating production operations maturity
- Showing service mesh and orchestration integration

---

## RAG Pipeline Architecture (Phase 8)

**File**: `rag-pipeline-architecture-phase8.png`  
**Created**: January 6, 2026  
**Updated**: January 8, 2026 (Phase 8 optimizations)  
**Source**: draw.io

**Shows 5-Layer RAG Pipeline**:
- **User Interaction Layer**: Streamlit chat UI, query input, conversation display
- **RAG Orchestration Layer**: Query processing, cache checking (semantic similarity 0.95), context assembly
- **Retrieval Layer**: Three-layer boosting system (Hybrid Search + Document Type + Topic Authority)
- **LLM Inference Layer**: Ollama (CPU/GPU hybrid), prompt construction, response generation
- **Storage Layer**: Embeddings (15MB JSON), conversations (SQLite WAL), analytics, semantic cache

**Three-Layer Boosting Architecture**:

**Layer 1 - Hybrid Search** (α=0.3):
- BM25 keyword search (30% weight)
- Vector semantic search (70% weight)
- Complementary strengths: keywords + semantics

**Layer 2 - Document Type Boosting**:
- ADRs: 1.5× boost (authoritative decisions)
- Reports: 1.3× boost (comprehensive content)
- Session notes: 0.8× reduction (working notes)
- Authority-based ranking

**Layer 3 - Topic Authority**:
- Title-based matching: 1.5× boost
- Precise query handling ("What is Nomad?" → ADR-001)
- Relevance amplification

**Performance Characteristics**:
- **Cache hit**: ~10ms (200× faster than full pipeline)
- **Cache miss**: ~2.5s (embedding + search + LLM)
- **Accuracy**: 90% overall (80% factual, 100% architectural)
- **Latency**: ~25ms search (production-grade)

**Optimization Journey**:
- Session 0: 0% baseline (vector-only search failed)
- Session 1: 20% (hybrid search added)
- Session 2: 75% (document boosting - breakthrough! +55pp)
- Session 6: 90% (topic authority - completion! +15pp)

**Data Flow Visualization**:
- Clear path from query to response
- Cache hit vs cache miss paths
- Component interactions visible
- Performance metrics at each stage

**Use For**:
- RAG architecture discussions
- Explaining optimization methodology (0% → 90%)
- Demonstrating systematic engineering
- Phase 8 breakthrough story

---

## Diagram Philosophy

**Visual Clarity**:
- Clean layouts with logical top-to-bottom flow
- Color-coded components by functional layer
- Clear labels and relationship arrows
- Professional quality suitable for presentations

**Detail Level**:
- **High-level**: System components and relationships (infrastructure diagram)
- **Mid-level**: Data flow and component interactions (RAG pipeline)
- **Low-level**: Documented in code, configs, and ADRs (not in diagrams)

**Security**:
- All IP addresses sanitized (generic private IPs)
- No real network topology exposed
- Port numbers shown (standard, non-sensitive)
- Component names visible (architectural value)

**Optimization**:
- Both diagrams optimized for web viewing
- High quality but reasonable file sizes (293-313KB)
- PNG format for universal compatibility
- Draw.io source files maintained in homelab-nomad project

---

## Usage Guidance

### **For Interviews**

**Infrastructure Architecture**:
- "Walk me through your infrastructure" → Show 10-layer stack
- "How do you handle service discovery?" → Point to Consul layer
- "Explain your observability approach" → Highlight Prometheus/Grafana

**RAG Pipeline Architecture**:
- "How did you optimize the RAG system?" → Show three-layer boosting
- "Explain your caching strategy" → Point to cache hit/miss paths
- "What was the breakthrough?" → Session 2 document boosting story

### **For Technical Discussions**

**System Design**:
- Infrastructure diagram shows complete production architecture
- Demonstrates understanding of layered system design
- Shows integration of multiple technologies

**Optimization Methodology**:
- RAG pipeline shows systematic approach
- Visualizes data-driven optimization
- Documents before/after improvements

### **For Portfolio Navigation**

**Complete Story**:
1. Infrastructure Overview presentation (Phases 4-6) → System context
2. Infrastructure Architecture diagram → Detailed topology
3. Phase 7 presentation → RAG platform creation
4. RAG Pipeline diagram → Component-level architecture
5. Phase 8 presentation → Optimization journey
6. Phase 8 journey narrative → Complete breakthrough story

---

## Diagram Maintenance

**Source Files**: Draw.io sources maintained in `/Users/mmazza/projects/homelab-nomad/docs/diagrams/`

**Version Control**: 
- Diagram updates tracked in homelab-nomad repository
- Portfolio copies updated when significant changes occur
- Change log maintained in homelab-nomad diagrams README

**Update Triggers**:
- Major architecture changes (new layers, services)
- Significant optimization improvements (Phase 9+)
- Technology stack evolution

**Sanitization Process**:
- All IP addresses replaced with generic private IPs (10.0.x.x)
- Network topology generalized
- Port numbers preserved (standard, educational value)
- Component relationships maintained

---

## Technical Details

### **Infrastructure Architecture**

**Technologies Visualized**:
- HashiCorp Nomad (3-node cluster orchestration)
- HashiCorp Consul (3-node service mesh)
- Traefik v2.10 (reverse proxy, auto-routing)
- Prometheus + Grafana (observability)
- Ollama (hybrid AI/ML: CPU + GPU)
- Synology NAS (NFS storage)
- 15+ application services

**Connections**:
- Solid lines: Traffic flow
- Dashed lines: Control/API communication
- Dotted lines: Monitoring/metrics
- Bold lines: GPU inference path
- Cyan: RAG platform components

### **RAG Pipeline Architecture**

**Components Detailed**:
- Streamlit UI: Port 8501, multi-turn conversations, source attribution
- Cache: Semantic similarity 0.95 threshold, 200× speedup
- Hybrid Search: BM25 (rank-bm25) + Vector (nomic-embed-text 768d)
- Document Boosting: Configurable multipliers per document type
- Topic Authority: Title matching with boost
- SQLite: WAL mode, 132KB database, 100% concurrency
- Embeddings: 15MB JSON, 611 chunks, 85 documents

**Performance Metrics**:
- Overall accuracy: 90%
- Search latency: ~25ms
- Cache hit rate: 40%
- LLM fallback rate: 40%

---

**Status**: 2 architecture diagrams (sanitized, documented)  
**Coverage**: Infrastructure (10 layers) + RAG Pipeline (5 layers + 3-layer boosting)  
**Quality**: Interview-ready, technically accurate, security-conscious
