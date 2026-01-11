# ADR-001: Orchestration Platform Selection

**Status**: ✅ APPROVED  
**Date**: December 15, 2025  
**Context**: Phase 2 - Orchestration Layer Implementation

---

## Decision

**Selected Platform**: HashiCorp Nomad

Deploy HashiCorp Nomad as the container orchestration platform for the homelab infrastructure.

---

## Context

### Infrastructure Overview
- 3 Proxmox VE hosts
- Container and VM workloads
- Learning/research environment
- Single-person operation
- Resource constraints (older hardware)

### Goals
1. Automate workload deployment and management
2. Enable efficient resource utilization across hosts
3. Simplify service lifecycle (deploy, scale, update, rollback)
4. Maintain operational simplicity
5. Support both stateful and stateless workloads
6. Learn modern orchestration patterns

---

## Options Considered

### Option 1: HashiCorp Nomad ✅ SELECTED

**Pros**:
- Simple single-binary deployment
- Low overhead (~50MB memory per node)
- Flexible workload support (containers, VMs, binaries)
- Gentle learning curve
- Resource efficient for older hardware
- HashiCorp ecosystem integration

**Cons**:
- Smaller ecosystem vs Kubernetes
- Less community content
- Lower enterprise adoption

**Operational Complexity**: Low  
**Resource Requirements**: Low (~200MB total)

---

### Option 2: Kubernetes (K3s)

**Pros**:
- Industry standard platform
- Massive ecosystem (Helm charts, operators)
- Highly transferable professional skills
- Rich feature set
- Strong community support

**Cons**:
- Steep learning curve
- Higher operational overhead
- Resource intensive (~1GB+ per node)
- Complex for homelab use case
- YAML verbosity

**Operational Complexity**: High  
**Resource Requirements**: Medium-High

---

### Option 3: Docker Swarm

**Pros**:
- Easiest setup
- Familiar Docker Compose syntax
- Low overhead (~100MB per node)
- Built into Docker

**Cons**:
- Declining adoption
- Limited ecosystem
- Uncertain long-term future
- Minimal professional relevance

**Operational Complexity**: Very Low  
**Resource Requirements**: Low

---

## Decision Criteria & Scoring

| Criterion | Weight | Nomad | K3s | Swarm |
|-----------|--------|-------|-----|-------|
| Operational Simplicity | 40% | 9/10 | 5/10 | 10/10 |
| Resource Efficiency | 25% | 9/10 | 6/10 | 9/10 |
| Learning Value | 20% | 6/10 | 10/10 | 3/10 |
| Ecosystem | 15% | 6/10 | 10/10 | 4/10 |
| **Weighted Total** | | **7.95** | **7.0** | **7.45** |

---

## Rationale

Nomad scored highest (7.95) when weighted against specific homelab needs:

### 1. Operational Simplicity Wins
Single binary deployment, minimal configuration, straightforward troubleshooting. More time for experimentation, less time fighting the orchestrator.

### 2. Resource Fit
~200MB total overhead vs ~3GB for K3s. Critical on older hardware with limited resources.

### 3. Flexibility Advantage
Nomad orchestrates containers, VMs, and raw binaries - fits diverse homelab workload profile without forcing everything into containers.

### 4. HashiCorp Ecosystem
Natural integration with Terraform, opens door to Consul (service mesh) and Vault (secrets) later.

### 5. Sufficient Learning Value
Teaches core orchestration concepts (scheduling, placement, health checks, service discovery) that translate to any platform.

### 6. Homelab Reality
Learning environment where simplicity and experimentation matter more than mimicking enterprise patterns exactly.

---

## Why Not K3s?

K3s scored 7.0, very close to Nomad. However:
- Kubernetes complexity (5/10 vs 9/10) would consume time better spent experimenting
- Resource overhead significant on older hardware
- Core orchestration concepts transfer; can migrate later with better hardware

**Reassessment Trigger**: If primary goal shifts to professional Kubernetes skills or hardware upgrade to 32GB+ RAM per node, revisit K3s.

---

## Consequences

### Positive
- **Low Operational Burden**: Simple to maintain and troubleshoot
- **Resource Efficient**: Maximum headroom for actual workloads
- **Flexible**: Handles diverse workload types naturally
- **Fast Learning**: Gentle curve enables rapid progress
- **Ecosystem Ready**: Foundation for Consul, Vault integration

### Neutral
- **Niche Skills**: Less directly transferable than Kubernetes
- **Smaller Community**: Fewer pre-built solutions available

### Negative (Mitigated)
- **Limited Ecosystem**: May need custom solutions vs using existing operators
- **Career Relevance**: Not industry-dominant like Kubernetes

---

## Alternatives Considered & Rejected

### Hybrid Approach (Proxmox + Orchestration)
- **Rejected**: Managing two orchestration layers adds complexity
- **Overhead**: VM overhead plus container overhead wasteful

### Full Kubernetes
- **Rejected**: Too resource-intensive and complex for homelab scale
- **Reconsider**: When hardware refreshes or professional K8s skills become primary goal

---

## Related Documentation

- **Journey**: Foundation for all subsequent infrastructure phases
- **Phase 4**: Service Discovery (Consul) builds on this decision
- **Phase 5**: Observability Stack deployment via Nomad

---

## Conclusion

HashiCorp Nomad provides the optimal balance of simplicity, resource efficiency, and learning value for a homelab environment. The decision prioritizes operational sustainability over enterprise pattern mimicry, enabling focus on workload experimentation rather than platform maintenance.

**Status**: ✅ APPROVED | **Implementation**: Phase 2 Complete | **Production**: 18+ services orchestrated

---

*"Simplicity is prerequisite for reliability." - Edsger W. Dijkstra*
