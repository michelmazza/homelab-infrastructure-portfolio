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

**Missing** (to be added):
- Phase 7 Streamlit RAG interface
- Phase 8 test results terminal
- Phase 9 MCP tools demo (when complete)

---

## Portfolio Value

These screenshots demonstrate:
- **Production Operations**: Full observability stack operational
- **Service Mesh**: Consul service registry and health checks
- **Service Discovery**: Automatic target discovery via Prometheus
- **Health Monitoring**: Comprehensive service health tracking
- **Orchestration**: 15+ services managed via Nomad
- **Reverse Proxy**: Dynamic routing with Traefik
- **SRE Practices**: Monitoring-first infrastructure

---

**Status**: 8 screenshots captured (Week 3 Session 1) ✅  
**Bonus**: Fixed Ollama monitoring during session (troubleshooting documented)  
**Next**: Phase 7/8 evidence screenshots, Phase 9 demos (when ready)
