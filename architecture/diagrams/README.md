# Architecture Diagrams

Visual representations of system architecture across phases.

---

## Infrastructure Architecture (Phase 6)

**File**: `infrastructure-architecture-phase6.png`  
**Created**: December 31, 2025  
**Updated**: January 12, 2026 (sanitized for portfolio)  

**Shows 9-Layer Stack (Pre-RAG)**:
- **DNS Layer**: Synology DNS (all services point to single IP)
- **Reverse Proxy Layer**: Traefik v2.10 with Consul catalog provider, host-based routing, Prometheus metrics
- **Service Discovery Layer**: Consul 3-node cluster (service registration, health monitoring, DNS interface)
- **Orchestration Layer**: Nomad 3-node cluster (container orchestration, job scheduling, resource allocation)
- **Application Services**: 15+ containerized services (web apps, search, infrastructure tools)
- **AI/ML Inference Layer**: Hybrid Ollama deployment (CPU cluster-based + GPU Mac Studio)
- **Observability Layer**: Prometheus (scraping), Grafana (dashboards), monitoring infrastructure
- **External GPU Infrastructure**: Mac Studio M1 Max (24× faster than CPU, GPU-accelerated inference)
- **Storage Layer**: Synology NAS via NFS (persistent data)

**Key Characteristics**:
- Foundation architecture before RAG platform
- 15+ services operational
- Hybrid AI/ML deployment (CPU learning + GPU production)
- Complete observability stack
- Production-grade infrastructure

**Use For**: 
- Showing infrastructure foundation
- Architecture discussions before RAG
- Demonstrating pre-RAG capabilities
- Baseline for Phase 7 comparison

---

## Infrastructure Architecture (Phase 7)

**File**: `infrastructure-architecture-phase7.png`  
**Created**: January 4, 2026  
**Updated**: January 11, 2026 (sanitized for portfolio)  

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

## Agentic RAG System Architecture (Phase 9)

**Files**: 
- `phase-9-system-topology.png` (System components and integration)
- `phase-9-data-flow.png` (Execution pipeline and Glass Box AI)

**Created**: January 16, 2026  
**Updated**: January 16, 2026 (Session 14)  

### System Topology

**Shows 5-Layer Agentic Architecture**:

**Layer 1 - User Interface**:
- Streamlit chat interface
- Multi-turn conversation support
- Glass Box AI transparency visualization

**Layer 2 - Agentic Reasoning**:
- Intent classifier (98.2% accuracy)
- Query analyzer and decomposition
- Tool selection strategy
- Response synthesis

**Layer 3 - MCP Tools (5 live tools)**:
- Prometheus API: Real-time metrics, health checks, alerts
- Nomad API: Job status, allocation data, cluster health
- Consul API: Service discovery, health checks, KV store
- ADR Threshold Injector: Decision criteria embedding (innovation!)
- Documentation RAG: Semantic + keyword search

**Layer 4 - Infrastructure Integration**:
- Prometheus: 14+ targets, 30s scrape interval
- Nomad: 3-node cluster orchestration
- Consul: Service mesh, health checks
- SQLite: Conversation persistence (WAL mode)

**Layer 5 - Storage & Knowledge**:
- RAG embeddings: 15MB, 611 chunks, 85 documents
- Conversation database: SQLite with analytics
- Infrastructure state: Live API queries (no caching!)

**Key Architecture Decisions**:
- **Live Infrastructure Queries**: Real-time state over caching (accuracy priority)
- **MCP Tool Pattern**: Standardized interface, consistent error handling
- **Glass Box AI**: Transparent reasoning and decision visualization
- **ADR Threshold Injector**: Embed decision criteria in tool responses (unique innovation!)

**Integration Characteristics**:
- Zero error rate (120 tests, 100% passing)
- 0.7s average tool response time
- 98.7% overall agentic accuracy
- 41-238× faster than target latency

### Data Flow Diagram

**Shows 8-Step Agentic Pipeline**:

**Step 1 - Query Reception**:
- User question captured via Streamlit
- Conversation context loaded
- Query preprocessing

**Step 2 - Intent Classification** (98.2% accuracy):
- Infrastructure query detection
- Documentation query detection
- Multi-step reasoning identification
- Tool selection strategy determined

**Step 3 - Tool Orchestration**:
- Single tool: Direct execution
- Multiple tools: Parallel coordination
- Cross-reference: Multi-tool validation

**Step 4 - MCP Tool Execution**:
- Prometheus: `query_metric`, `query_range`
- Nomad: `query_job_status`, `query_allocations`
- Consul: `query_service`, `query_health`
- ADR Injector: Threshold-aware responses
- Documentation RAG: Hybrid search retrieval

**Step 5 - Cross-Reference Validation** (100% accuracy):
- Multi-tool consistency checking
- Threshold comparison (ADR-injected criteria)
- Contradiction detection
- Confidence scoring

**Step 6 - Response Synthesis**:
- Combine tool results
- Apply LLM reasoning
- Format for clarity
- Add source attribution

**Step 7 - Glass Box AI Visualization**:
- Show reasoning chain
- Display tool calls and responses
- Highlight decision points
- Expose confidence scores

**Step 8 - Delivery**:
- Natural language response
- Transparent reasoning visible
- Sources cited
- Follow-up suggested

**Performance Pipeline Metrics**:
- Intent classification: 21ms average
- Tool execution: 0.7s average (dominated by API calls)
- Cross-reference: 121ms average
- Overall latency: 41-238× better than target
- Glass Box rendering: <50ms overhead

**Innovation Highlights**:

**ADR Threshold Injector** (Week 1):
- Embeds decision thresholds directly in tool responses
- Enables single-step diagnostic reasoning
- Example: "CPU > 80% threshold (ADR-008)" in metrics response
- Transforms multi-step into single-step analysis

**Glass Box AI** (Week 3):
- Transparent reasoning vs black box models
- Visual decision chain exposure
- User trust through explainability
- Debugging and validation simplified

**Cross-Reference Engine** (100% accuracy):
- Multi-tool validation for critical queries
- Prometheus + Nomad health cross-check
- ADR threshold comparison across tools
- Contradiction detection and resolution

**Use For**:

**System Topology Diagram**:
- "Explain your agentic RAG architecture" → Show 5-layer integration
- "How do you handle live infrastructure queries?" → Point to MCP tools layer
- "What makes this different from standard RAG?" → Highlight ADR Threshold Injector

**Data Flow Diagram**:
- "Walk me through an infrastructure query" → Show 8-step pipeline
- "How do you ensure accuracy?" → Point to cross-reference validation (100%)
- "What's Glass Box AI?" → Highlight transparency in Step 7

**Together**:
- Complete picture: topology (components) + data flow (execution)
- Demonstrates systematic architecture thinking
- Shows innovation in both design and implementation

---

## Observation to Autonomy (Phase 22-23) ⭐ NEW

**File**: `phase-22-23-observation-to-autonomy.png`
**Created**: May 2026
**Updated**: May 2026 (sanitized for portfolio)

**Shows the Phase 22 dry-run foundation feeding the Phase 23 5-gate live execution pipeline**:
- **Phase 22 zone** (left, amber): allowlist matcher → enrichment → dry-run executor (Levels 0+1)
- **Phase 23 spine** (center, cyan): 5 gates in series — kill-switch → rate-limit → pre-flight → restart → stable-duration → audit
- **Three independent safety controls** (right, green): kill-switch (operator intent, no audit row), rate-limit (3 actions per metric/node per 10 min), loop guard (structural — every audit row counts toward the rate-limit COUNT)
- **LLM boundary** (bottom, pink annotation): no LLM at any gate; all checks deterministic
- **Rollback loop** (single-strike): any failure during the 60s stable-duration window → rolled_back

**Key Architecture Decisions**:
- ADR-025: 5-gate pipeline rationale
- ADR-026: Kill-switch confirmation gate placement
- ADR-027: Three-Layer UI Validation standard

**Use For**:
- "How do you safely execute remediation actions?" → Show the 5-gate spine
- "Where's the LLM in the execution path?" → Point to the bottom annotation: nowhere
- "How is the kill-switch different from the rate-limit?" → Operator intent (no row) vs automatic safety (row written, outcome=rate_limited)

---

## Distributed System Topology (Phase 21)

**File**: `phase-21-system-topology.png`
**Created**: April 2026
**Updated**: April 2026 (sanitized for portfolio)

**Shows 4-Node Cluster with Distributed Data Plane**:
- **Mac Studio (Development)**: Streamlit RAG app + Ollama LLM inference — not managed by Nomad; node eligibility disabled for scheduling
- **lab-node-1 (ARM64)**: prometheus-scraper (docker driver) — writes metrics to PostgreSQL on lab-node-3
- **lab-node-2**: node-exporter (docker driver) — metrics collection
- **lab-node-3 (AMD64)**: PostgreSQL, registry:2, log-event-collector, observability-lifecycle, Grafana, Traefik, Searxng, node-exporter

**8 Data Flows Visualized**:
1. Prometheus scraper (lab-node-1) → PostgreSQL (lab-node-3): cross-node metric writes
2. Log event collector → PostgreSQL: Nomad/Consul event ingestion
3. Observability lifecycle (periodic): dual-table retention (metric_snapshots + log_events)
4. Alertmanager → enrichment webhook → top-cause annotation
5. Correlation engine → rag/db.py factory → PostgreSQL read path
6. Container registry (lab-node-3:5000) → image pull on any cluster node
7. Nomad scheduler → placement via job-file constraints (ADR-020)
8. RAG app (Mac Studio) → PostgreSQL (lab-node-3): read queries via rag/db.py

**Key Architecture Decisions**:
- **No Mac Studio in compute plane**: node eligibility disabled; zero Nomad jobs scheduled
- **Shared image pattern**: one Dockerfile, three entrypoints (scraper / lifecycle / collector)
- **ADR-020 placement policy**: macOS-exclusion constraint encodes architectural intent
- **Scoped IAM**: `app_role` (CRUD only) replaces SUPERUSER for all three services

**Network Details**:
- All IPs sanitized (generic private IPs)
- Cross-node writes proven: ARM64 → AMD64 (lab-node-1 → lab-node-3)
- Container registry on lab-node-3 serves multi-arch images (amd64 + arm64)

**Use For**:
- Distributed systems and data plane migration discussions
- "How do you handle cross-node writes?" → Show scraper → PostgreSQL flow
- "What's your containerization strategy?" → Shared-image pattern + placement constraints
- ADR-020 operational-state vs architectural-intent discussion

---

## Performance & Intelligence Architecture (Phase 10)

**Files**: 
- `phase-10-system-topology.png` (System components with CoVe integration)
- `phase-10-cove-detail.png` (7-component verification flow)

**Created**: January 24, 2026  
**Updated**: February 2, 2026  

### System Topology

**Shows 5-Layer Architecture with Performance Optimization**:

**Layer 1 - User Interface**:
- Streamlit chat interface
- Glass Box AI with enhanced verification display
- Confidence heatmap visualization

**Layer 2 - Chain of Verification (CoVe)**:
- 7-component tiered verification system
- Text match (0.5ms) → Semantic (2ms) → LLM fallback (50ms)
- Multi-factor confidence scoring (60/30/10 split)

**Layer 3 - Performance Infrastructure**:
- TTL cache with jitter (605,000× speedup)
- Connection pooling (99% reuse rate)
- Parallel MCP execution (2.8× faster)

**Layer 4 - Agentic Tools**:
- 5 MCP tools with ADR Threshold Injector
- Prometheus, Nomad, Consul APIs
- Documentation RAG with boosted retrieval

**Layer 5 - Storage & Knowledge**:
- SQLite with WAL mode
- Embedding cache (15MB, 611 chunks)
- Semantic similarity cache (0.95 threshold)

### CoVe Detail Diagram

**Shows 7-Component Verification Pipeline**:

1. **Claim Extraction**: Parse response into verifiable claims
2. **Text Match**: Exact substring verification (0.5ms)
3. **Semantic Match**: Embedding similarity check (2ms)
4. **LLM Verification**: Context-aware validation (50ms)
5. **Multi-Factor Scoring**: Combine verification results
6. **Confidence Aggregation**: Weighted claim confidence
7. **Glass Box Visualization**: Display verification status

**Key Metrics**:
- 83× faster than uniform LLM verification
- 3ms average CoVe overhead (94% better than 50ms target)
- 18% fallback rate (beat <20% target by 2pp)

**Use For**:
- Explaining verification architecture
- Performance optimization discussions
- CoVe design interviews

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

**Agentic RAG Architecture**:
- "What's your most innovative project?" → Phase 9 agentic system
- "How do you integrate AI with infrastructure?" → Show MCP tools layer
- "Explain Glass Box AI" → Data flow diagram, Step 7 transparency
- "What makes your RAG system unique?" → ADR Threshold Injector innovation

### **For Technical Discussions**

**System Design**:
- Infrastructure diagram shows complete production architecture
- Demonstrates understanding of layered system design
- Shows integration of multiple technologies

**Optimization Methodology**:
- RAG pipeline shows systematic approach
- Visualizes data-driven optimization
- Documents before/after improvements

**Agentic Systems**:
- Phase 9 topology shows component integration
- Data flow demonstrates multi-step reasoning
- Glass Box AI shows transparency-first design
- Cross-reference validation shows accuracy priority

### **For Portfolio Navigation**

**Complete Story**:
1. Infrastructure Overview presentation (Phases 4-6) → System context
2. Infrastructure Architecture diagram → Detailed topology
3. Phase 7 presentation → RAG platform creation
4. RAG Pipeline diagram → Component-level architecture
5. Phase 8 presentation → Optimization journey
6. Phase 8 journey narrative → Breakthrough story
7. Phase 9 presentation → Agentic transformation
8. Phase 9 system topology → Architecture deep-dive
9. Phase 9 data flow → Execution pipeline detail
10. Phase 9 journey narrative → Complete implementation story
11. Phase 9 metrics page → Quantified results

---

## Diagram Maintenance

**Update Triggers**:
- Major architecture changes (new layers, services)
- Significant optimization improvements (Phase 9+)
- Technology stack evolution

**Sanitization Process**:
- All IP addresses replaced with generic private IPs (10.0.x.x)
- Network topology generalized
- Port numbers preserved (standard, educational value)
- Component relationships maintained
- **Embedded editor source stripped**: diagram PNGs exported from draw.io carry the full
  diagram source in a PNG `tEXt`/`zTXt`/`iTXt` metadata chunk. That source is invisible in the
  rendered image but trivially extractable, and it is *not* covered by sanitizing what the
  diagram displays. All `tEXt`/`zTXt`/`iTXt` chunks are therefore removed from every published
  diagram PNG (image data left byte-identical — no recompression). Editable `.drawio` sources
  stay in the private repository; this directory publishes rendered PNGs only.

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

### **Agentic RAG Architecture**

**Components Detailed**:
- Intent Classifier: 98.2% accuracy (54/55 tests)
- MCP Tools: 5 tools, 120 tests, 100% success rate
- Cross-Reference: 100% accuracy (60/60 tests)
- ADR Threshold Injector: Embedded decision criteria
- Glass Box AI: Transparent reasoning visualization
- SQLite: Conversation persistence, analytics

**Performance Metrics**:
- Overall agentic accuracy: 98.7% (157/159 tests)
- Response time: 21-121ms (41-238× faster than target)
- Tool execution: 0.7s average (99.99% API call time)
- Zero error rate: 120/120 tool tests passing

**Innovation Metrics**:
- ADR Threshold Injector: +10pp accuracy improvement
- Glass Box AI: Enhanced user trust, simplified debugging
- Cross-reference validation: 100% accuracy on critical queries

---

**Status**: 9 architecture diagrams (sanitized, documented)
**Coverage**: Infrastructure (Phases 6-7) + RAG Pipeline (Phase 8) + Agentic RAG (Phase 9) + Performance (Phase 10) + Distributed System Topology (Phase 21) + Observation to Autonomy (Phase 22-23)
**Quality**: Interview-ready, technically accurate, security-conscious
