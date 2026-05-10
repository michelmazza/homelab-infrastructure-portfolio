# Screenshots

Visual evidence of production infrastructure and observability stack.

---

## Observability Stack

### Grafana Dashboards Overview
**File**: `grafana_dashboards.png`  
**Shows**: Dashboard management interface with 5 configured dashboards including Host Overview, Ollama Monitoring, Request Flow Analysis, and Service Health Overview. Tags: ai, llm, ollama.

### Host Overview Dashboard
**File**: `dashboard_host_overview.png`  
**Shows**: Complete infrastructure monitoring dashboard tracking Nomad cluster health, resource utilization across 3 nodes (lab-node-1, lab-node-2, lab-node-3), and system metrics.

### Service Health Overview Dashboard
**File**: `dashboard_service_health_overview.png`  
**Shows**: Comprehensive service health monitoring across all deployed services, health check status, and service availability metrics.

### Ollama Monitoring Dashboard
**File**: `ollama_monitoring_dashboard.png`  
**Shows**: AI/ML platform monitoring with Ollama endpoint health tracking for both CPU (Nomad cluster) and GPU (ML workstation) instances. Demonstrates blackbox exporter probes for HTTP health checks, response times, and availability metrics.

### Prometheus Targets
**File**: `prometheus_targets.png`  
**Shows**: Prometheus monitoring targets including node-exporter (3/3 up), nomad (3/3 up), prometheus self-monitoring, traefik (1/1 up), and ollama-health (2/2 up). Full observability stack with service discovery and health checks.

---

## Infrastructure Services

### Consul Services Dashboard
**File**: `consul_dashboard_services.png`  
**Shows**: HashiCorp Consul service mesh with registered services, health checks, and service discovery.

### Nomad Dashboard
**File**: `nomad_dashbord.png`  
**Shows**: HashiCorp Nomad orchestration platform with job management, allocation status, and cluster overview. Running 15+ containerized services across the cluster.

### Traefik Dashboard
**File**: `traefik_dashboard.png`  
**Shows**: Traefik reverse proxy dashboard with HTTP routers, services, and middleware configuration. Demonstrates dynamic service discovery and automatic routing.

---

## RAG Platform User Interface (Phase 7-8)

### Streamlit Welcome Interface
**File**: `streamlit_ui_welcome.png`

Production RAG system showing system status (611 chunks, 85 documents), clean conversational UI, and professional dark theme.

### Streamlit Query Results & RAG Workflow
**File**: `streamlit_ui_query_results.png`

Complete RAG workflow demonstrating 90% accuracy — hybrid search (BM25+Vector), document authority boosting (ADR=1.5×), topic authority, and real confidence scores. Visual proof the Phase 8 optimization is working.

---

## Phase 9: Glass Box AI — Visual Transparency

### Glass Box AI - Thought Trace
**File**: `phase-9-glass-box-thought-trace.png`

Step-by-step reasoning transparency. Query: "Is Traefik running?" — shows intent classification (LIVE_STATUS, 90.0% confident), tool calls, and timing breakdown. Makes AI reasoning completely transparent.

### Glass Box AI - Timeline Visualization
**File**: `phase-9-glass-box-timeline.png`

Plotly Gantt chart showing execution timeline. Key insight: 99.99% of time in MCP API calls, not AI reasoning — guided optimization focus.

### Glass Box AI - Performance Dashboard
**File**: `phase-9-glass-box-performance.png`

Performance metrics and speedup visualization — actual (1010ms) vs target (5000ms). **5× faster than target.**

---

## Phase 10: Chain of Verification (CoVe)

### Glass Box AI - Enhanced Verification Display
**File**: `phase-10-glass-box-tab1-reasoning.png`

Enhanced Glass Box showing CoVe results — verification status per claim, multi-factor confidence scores, source attribution.

### Glass Box AI - Verification Detail
**File**: `phase-10-glass-box-tab2-verification.png`

Tiered verification breakdown: text match (0.5ms) → semantic (2ms) → LLM fallback (50ms). 83× speedup over uniform LLM verification.

### Glass Box AI - Confidence Dashboard
**File**: `phase-10-glass-box-tab3-confidence.png`

Multi-factor confidence scoring: verification (60%), source quality (30%), response length (10%). Complete transparency into confidence calculation.

---

## Phase 17 — Advanced Synthesis

| File | Description |
|---|---|
| `phase-10-glass-box-before-3tabs.png` | Glass Box UI before Phase 17 — original 3 tabs (baseline for comparison) |
| `phase-17-glass-box-conflicts-tab.png` | Tab 4: Conflicts — 7-category detection results with consensus strength |
| `phase-17-glass-box-resolution-low-consensus.png` | Tab 5: Resolution — weighted scoring + resolution strategy |
| `phase-17-glass-box-provenance-tab.png` | Tab 6: Provenance — full fact lineage chains |

**Achievement**: Glass Box expanded from 3 → 6 tabs. 0.208ms conflict resolution (48× under 10ms budget).

---

## Phase 18 — Predictive Diagnostics

| File | Description |
|---|---|
| `phase-18-tab7-smoke-test-full.png` | Glass Box Tab 7 — all 9 metric rows, R² color-coding, observer 🔴 indicators |
| `phase-18-tab7-smoke-test-query1.png` | Full app view during active diagnostic query |
| `phase-18-tab7-smoke-test-query2.png` | Predictive Diagnostics + conflict detected |
| `phase-18-tab7-smoke-test-query3.png` | Predictive Diagnostics + no conflict baseline |

**Achievement**: Zero-LLM OLS engine at ~0.05ms P95. Self-aware observer effect: RAG query intensity correlated with own resource footprint.

---

## Phase 19 — Cross-Modal Diagnostic Correlation

| File | Description |
|---|---|
| `phase-19-tab7-top-cause-candidate.png` | Tab 7: Top Cause column — Candidate state with score tooltip |
| `phase-19-tab7-unknown-cause.png` | Tab 7: "Unknown Cause" state (events exist, no correlation above threshold) |
| `phase-19-tab7-no-events-logged.png` | Tab 7: "No Events Logged" state (collector gap) |
| `phase-19-tab7-collector-gap.png` | Tab 7: "Collector Gap" state — fourth diagnostic state |

**Achievement**: Four-state "Quiet Period Paradox" resolution. "Why is CPU high?" answered in <10ms with zero LLM in the analytical path.

---

## Phase 20 — Reliability & Resonance

| File | Description |
|---|---|
| `phase-20-glass-box-all-tabs.png` | Full 8-tab Glass Box UI — complete Conversational AIOps interface |
| `phase-20-tab4-no-conflicts.png` | Tab 4: Synthesis Conflicts — "No conflicts detected across sources" |
| `phase-20-tab5-resolution.png` | Tab 5: Synthesis Resolution panel |
| `phase-20-tab6-provenance.png` | Tab 6: Provenance — 127 unique facts from 3 sources, 5.07ms overhead |
| `phase-20-tab7-predictive-diagnostics.png` | Tab 7: Predictive Diagnostics — 9 metrics, R² color-coding, Top Cause column |
| `phase-20-tab7-no-events-logged.png` | Tab 7: "No Events Logged" state (4th diagnostic state) |
| `phase-20-tab8-empty-state.png` | Tab 8: Diagnostic Thread — empty state "No diagnostic investigations this session" |

**Achievement**: 8-tab Glass Box. Tab 8 Diagnostic Thread: append-only session investigation timeline. Verifiable Inference: follow-up responses append live trend status. System maintains the diagnostic thread over time.

---

## Coverage Summary

**Observability** (8 screenshots):
- Grafana dashboards, Prometheus targets, host metrics, service health

**Infrastructure** (3 screenshots):
- Consul service mesh, Nomad orchestration, Traefik reverse proxy

**AI/ML Platform** (2 screenshots):
- Streamlit RAG interface (welcome + query workflow with 90% accuracy visible)

**Glass Box AI — 7 tabs evolution**:
- Phase 9: Original 3 tabs (Thought Trace, Timeline, Performance)
- Phase 10: CoVe verification (3 screenshots)
- Phase 17: Synthesis tabs 4–6 (4 screenshots, +before baseline)
- Phase 18: Predictive Diagnostics Tab 7 (4 screenshots)
- Phase 20: Full 8-tab UI (7 screenshots) ⭐ **LATEST**

**Phase 19** (4 screenshots):
- Four-state Top Cause column (Candidate, Unknown Cause, No Events, Collector Gap)

---

**Status**: 31 screenshots total ✅  
**Latest**: Phase 20 — 8-tab Glass Box UI, Tab 8 Diagnostic Thread, Verifiable Inference  
**Coverage**: Infrastructure → AI/ML → Explainable AI → Verified AI → Synthesis → Predictive → Correlative → Conversational AIOps  
**Grade**: Portfolio-ready ✅
