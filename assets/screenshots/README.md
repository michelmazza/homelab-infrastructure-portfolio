# Screenshots

Visual evidence of production infrastructure and observability stack.

---

## Observability Stack

### Grafana Dashboards Overview
**File**: `grafana_dashboards.png`  
**Shows**: Dashboard management interface with 5 configured dashboards including Host Overview, Ollama Monitoring, Request Flow Analysis, and Service Health Overview. Tags: ai, llm, ollama.

### Host Overview Dashboard
**File**: `dashboard_host_overview.png`  
**Shows**: Complete infrastructure monitoring dashboard tracking Nomad cluster health, resource utilization across 3 nodes (lab01, lab02, lab03), and system metrics.

### Service Health Overview Dashboard
**File**: `dashboard_service_health_overview.png`  
**Shows**: Comprehensive service health monitoring across all deployed services, health check status, and service availability metrics.

### Ollama Monitoring Dashboard
**File**: `ollama_monitoring_dashboard.png`  
**Shows**: AI/ML platform monitoring with Ollama endpoint health tracking for both CPU (Nomad cluster) and GPU (Mac Studio) instances. Demonstrates blackbox exporter probes for HTTP health checks, response times, and availability metrics. Shows hybrid Ollama deployment architecture.

**Achievement**: Fixed blackbox-exporter endpoint configuration during Week 3 Session 1, demonstrating systematic troubleshooting and monitoring skills.

### Prometheus Targets
**File**: `prometheus_targets.png`  
**Shows**: Prometheus monitoring targets including:
- **node-exporter** (3/3 up): Host metrics from all 3 Proxmox nodes
- **nomad** (3/3 up): Nomad cluster metrics from all servers
- **prometheus** (1/1 up): Self-monitoring
- **traefik** (1/1 up): Reverse proxy metrics
- **ollama-health** (2/2 up): Ollama endpoints (CPU + GPU) ✅ Fixed!

**Demonstrates**: Full observability stack with service discovery, health checks, and multi-target monitoring.

---

## Infrastructure Services

### Consul Services Dashboard
**File**: `consul_dashboard_services.png`  
**Shows**: HashiCorp Consul service mesh with registered services, health checks, and service discovery. Demonstrates service registry and health monitoring across the cluster.

### Nomad Dashboard
**File**: `nomad_dashbord.png`  
**Shows**: HashiCorp Nomad orchestration platform with job management, allocation status, and cluster overview. Running 15+ containerized services across 3-node cluster.

### Traefik Dashboard
**File**: `traefik_dashboard.png`  
**Shows**: Traefik reverse proxy dashboard with HTTP routers, services, and middleware configuration. Demonstrates dynamic service discovery and automatic routing.

---

## RAG Platform User Interface (Phase 7-8)

### Streamlit Welcome Interface
**File**: `streamlit_ui_welcome.png`

Production RAG system showing:
- **System Status**: 611 chunks loaded from 85 documents
- **Interface**: Clean conversational UI ("HomeLab Knowledge Chat")
- **Navigation**: Sidebar with conversation history and controls
- **Design**: Professional dark theme with clear information hierarchy

**Context**: Built in Phase 7, this interface serves as the entry point for the production RAG system optimized in Phase 8. The system loads the complete knowledge base (611 chunks from 85 documents) and provides a ChatGPT-like experience for querying homelab documentation.

---

### Streamlit Query Results & RAG Workflow
**File**: `streamlit_ui_query_results.png`

Complete RAG workflow demonstrating the 90% accuracy achievement in action:

**Query Example**: "What is Nomad?"

**Retrieved Sources** (Top 3 with confidence scores):
1. **ADR-001-orchestration-platform-selection.md** (score: 0.957) 🚀
   - Authoritative document boost working perfectly!
2. **Phase-6-Completion-Report.md** (score: 0.917)
   - Topic match boost applied (title contains "Nomad")
3. **ADR-001-Phase-6-AI-ML-Foundation.md** (score: 0.870)
   - Topic match boost applied

**Search Configuration** (visible in left sidebar):
- **Mode**: Hybrid + Boosting (BM25 + Vector)
- **α = 0.3**: 30% BM25 (keywords), 70% Vector (semantics)
- **Boost Factors**:
  - ADR: 1.5× (authoritative documents)
  - Topic Match: 1.5× (title contains query term)
  - Reports: 1.3× (formal documentation)
  - Session Notes: 0.8× (de-boost exploratory content)
  - ADR-Index: 0.7× (table of contents)

**Performance Metrics**:
- **Query Cache**: 0% hit rate (fresh query)
- **Analytics**: 2 queries processed, 0% fallback rate
- **Response Time**: ~1497ms average (includes LLM generation)

**LLM Response**: Accurate, comprehensive answer citing the correct authoritative source (ADR-001)

**Technical Achievement**: 
- Without optimization: ADR-001 ranked #8 (wrong!)
- With Phase 8 optimization: ADR-001 ranked #1 with 0.957 confidence (correct!)
- Demonstrates calculated 1.5× boost factor working exactly as designed
- Visual proof of 90% accuracy optimization success

**What This Shows**:
- ✅ Hybrid search combining keywords + semantics
- ✅ Document type boosting prioritizing authoritative sources
- ✅ Topic authority matching for precise queries
- ✅ Complete RAG pipeline: query → retrieval → generation → response
- ✅ Professional UI with technical depth (config visible)
- ✅ Real confidence scores showing optimization impact

---

## Coverage Summary

**Observability**:
- ✅ Grafana (5 dashboards configured)
- ✅ Prometheus (multiple targets monitored)
- ✅ Host metrics (3 nodes)
- ✅ Service health monitoring

**Infrastructure**:
- ✅ Consul service mesh (service registry)
- ✅ Nomad orchestration (15+ services)
- ✅ Traefik reverse proxy (dynamic routing)
- ✅ Service discovery operational

**AI/ML Platform**:
- ✅ Streamlit RAG interface (welcome + query workflow)
- ✅ 90% accuracy optimization visible in action
- ✅ Complete retrieval pipeline with scoring
- ✅ Professional user experience

**Future Additions**:
- Phase 9 storyboard (Session 13)
- Phase 9 architecture diagrams (Session 14)
- Integration chaos testing results (future)

---

## Phase 9: Glass Box AI - Visual Transparency (Week 3)

### Glass Box AI - Thought Trace
**File**: `phase-9-glass-box-thought-trace.png`

Explainable AI in action showing step-by-step reasoning.

**Query**: "Is Traefik running?"

**Visual Elements**:
- Intent: LIVE_STATUS (90.0% confident)
- Total Time: 1010ms | Steps: 3 | Success: 100%
- Expandable step details with JSON inputs/outputs

**Steps Shown**:
- 🎯 Step 1: Classify intent (0.1ms)
- 🔧 Step 2: Get traefik status (1009.8ms) ← Bottleneck!
- ✨ Step 3: Synthesize answer (0.0ms)

**Achievement**: Makes AI reasoning completely transparent. Users see what intent was detected, which tools were called, what data was used, and where time was spent.

---

### Glass Box AI - Timeline Visualization  
**File**: `phase-9-glass-box-timeline.png`

Plotly Gantt chart showing execution timeline.

**Key Insight**: 
- Intent: 0.1ms (0.01%)
- MCP tools: 1009.8ms (99.99%) ← Optimization target
- Synthesis: 0.0ms (0.00%)

**Impact**: Visual proof that 99.99% of time is in MCP API calls, NOT AI reasoning. Guides optimization focus.

---

### Glass Box AI - Performance Dashboard
**File**: `phase-9-glass-box-performance.png`

Performance metrics and speedup visualization.

**Left**: Pie chart showing time breakdown  
**Right**: Bar chart comparing actual (1010ms) vs target (5000ms)

**Result**: **5× faster than target!** 🏆

**Demonstrates**: Both performance achievement AND engineering discipline (bottleneck identification).

---

## Phase 10: Chain of Verification (CoVe) Screenshots (January 2026)

### Glass Box AI - Enhanced Verification Display
**File**: `phase-10-glass-box-tab1-reasoning.png`

Enhanced Glass Box AI showing Chain of Verification results.

**Visual Elements**:
- Verification status per claim (verified/unverified/partial)
- Multi-factor confidence scores (60/30/10 breakdown)
- Source attribution with verification evidence
- Tiered verification pathway indicator

**Achievement**: Makes verification process completely transparent. Users see which claims were verified by text match, semantic similarity, or LLM fallback.

---

### Glass Box AI - Verification Detail
**File**: `phase-10-glass-box-tab2-verification.png`

Detailed verification breakdown showing CoVe in action.

**Key Insight**:
- Text match: 0.5ms (fastest, most certain)
- Semantic: 2ms (good balance)
- LLM fallback: 50ms (highest accuracy, slowest)

**Impact**: Visual proof of tiered verification working. Shows 83× speedup over uniform LLM verification.

---

### Glass Box AI - Confidence Dashboard
**File**: `phase-10-glass-box-tab3-confidence.png`

Multi-factor confidence scoring visualization.

**Scoring Breakdown**:
- Verification factor: 60% weight
- Source quality: 30% weight
- Response length: 10% weight

**Result**: Complete transparency into how confidence is calculated, not just the final number.

**Demonstrates**: Both accuracy improvement AND engineering discipline (explainable scoring).

---


## Phase 18 — Predictive Diagnostics (March 2026)

### Glass Box Tab 7 — Predictive Diagnostics

| File | Description |
|---|---|
| `phase-18-tab7-smoke-test-full.png` | Glass Box Tab 7, all 9 metric rows, R² color-coding, observer 🔴 indicators |
| `phase-18-tab7-smoke-test-query1.png` | Full app view during active query |
| `phase-18-tab7-smoke-test-query2.png` | Predictive Diagnostics + conflict detected |
| `phase-18-tab7-smoke-test-query3.png` | Predictive Diagnostics + no conflict |

---

## Phase 17 — Advanced Synthesis (March 2026)

| File | Description |
|---|---|
| `phase-10-glass-box-before-3tabs.png` | Glass Box UI before Phase 17 — original 3 tabs |
| `phase-17-glass-box-conflicts-tab.png` | Conflicts tab — 7-category detection results |
| `phase-17-glass-box-resolution-low-consensus.png` | Resolution tab — weighted scoring + consensus strength |
| `phase-17-glass-box-provenance-tab.png` | Provenance tab — full fact lineage chains |

---

## Portfolio Value

These screenshots demonstrate:
- **Production Operations**: Full observability stack operational
- **Service Mesh**: Consul + Nomad + Traefik orchestration
- **AI/ML Engineering**: 90% RAG accuracy + Agentic capabilities
- **Explainable AI**: Glass Box visualization ⭐ NEW!
- **Full-Stack Development**: Backend + frontend + visualization  
- **SRE Practices**: Monitoring-first infrastructure
- **Innovation**: Visual transparency in AI reasoning ⭐

---

**Status**: 24 screenshots total ✅ (+4 Phase 18)

**Week 3 Session 1**: Infrastructure + observability (8 screenshots)

**Week 3 Session 4**: RAG platform UI (2 screenshots)

**Week 3 Session 12**: Glass Box AI (3 screenshots)

**Phase 10**: CoVe verification (3 screenshots)

**Phase 17**: Advanced Synthesis — Glass Box 6 tabs (4 screenshots)

**Phase 18**: Predictive Diagnostics — Glass Box Tab 7 (4 screenshots) ⭐ NEW!

**Coverage**: Complete system from infrastructure → AI/ML → explainable AI → verified AI → synthesis → predictive

**Grade**: Portfolio-ready ✅
