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
- Phase 9 MCP tools integration demo (when complete)
- Agentic RAG workflow screenshots
- Multi-step reasoning examples

---

## Portfolio Value

These screenshots demonstrate:
- **Production Operations**: Full observability stack operational
- **Service Mesh**: Consul service registry and health checks
- **Service Discovery**: Automatic target discovery via Prometheus
- **Health Monitoring**: Comprehensive service health tracking
- **Orchestration**: 15+ services managed via Nomad
- **Reverse Proxy**: Dynamic routing with Traefik
- **AI/ML Engineering**: Production RAG system with 90% accuracy
- **Full-Stack Development**: Backend optimization + professional UI
- **SRE Practices**: Monitoring-first infrastructure

---

**Status**: 10 screenshots total ✅  
**Week 3 Session 1**: Infrastructure + observability (8 screenshots)  
**Week 3 Session 4**: RAG platform UI (2 screenshots)  
**Coverage**: Complete system from infrastructure → observability → AI/ML → user experience  
**Next**: Phase 9 agentic workflows (when complete)
