# ADR-020: Nomad-Native Discovery + Job-File Placement Policy for Heterogeneous Clusters

**Status**: Accepted
**Date**: 2026-04-12
**Phase**: 21 — Distributed Foundations
**Deciders**: Michel Mazza

---

## Context

### The Incident

Phase 21 containerized the first Docker-based Nomad job in the cluster.
Deploying a new container image required restarting Docker on one node,
which forced the Nomad scheduler to re-evaluate every job pinned to that
node. This rescheduling event exposed a latent infrastructure gap: the node
had no Consul agent installed, and every job with a `service {}` block
(which requires Consul for service registration) entered a permanent
pending state.

The jobs had been running successfully for weeks. The `service {}` blocks
were non-functional but harmless — Nomad only validates service registration
requirements at scheduling time, not continuously. The Docker restart forced
a re-evaluation that surfaced the gap.

### The Heterogeneous Cluster Problem

The cluster contains four nodes: three Linux servers and one macOS workstation
(Mac Studio). After containerizing workloads with Linux Docker images, a
placement question emerged: what prevents Nomad from scheduling a Linux
container on the macOS host?

The initial approach — marking the macOS node as ineligible via an
operational command — works but is ephemeral state. It survives until someone
re-enables the node, is not version-controlled, and applies cluster-wide
rather than per-job.

---

## Decision 1: Nomad-Native Service Discovery for Consul-Less Nodes

Remove `service {}` blocks from all jobs pinned to the Consul-less node.
Nomad-native service discovery is sufficient for these services because:

- No consumer resolves them via Consul DNS
- No service mesh (Consul Connect) is in use
- Each service is pinned to a single node (no load balancing)
- Nomad provides its own health checking via task-level `check {}` blocks

Consul remains the authoritative service discovery mechanism for the two
nodes where it is installed. This decision documents a node-specific
exception, not a deprecation of Consul.

---

## Decision 2: Job-File Constraints as Durable Placement Policy [PRIMARY]

Any Nomad job using a Linux-only container image must carry an explicit
constraint block excluding the macOS node in the job file.

### The Operational vs Architectural Distinction

This is the core insight. Two mechanisms exist for controlling job placement,
and they serve fundamentally different purposes:

| Mechanism | Type | Scope | Durability | Who controls it |
|-----------|------|-------|------------|-----------------|
| Node eligibility command | Operational state | Cluster-wide | Ephemeral — survives until anyone reverses it | Anyone with cluster access |
| Job-file constraint block | Architectural intent | Job-specific | Durable — committed to git, survives cluster resets | Git history |

**Operational state** answers: "Is this node currently accepting work?"
**Architectural intent** answers: "Should this specific job ever run on this node?"

### Why Operational State Alone Is Insufficient

The failure mode without job-file constraints:

1. macOS node is marked ineligible (operational state)
2. Linux containers run correctly on the three Linux nodes
3. Months later, someone re-enables the macOS node for a different purpose
4. On the next rescheduling event (node failure, rolling update), the
   macOS node is a valid placement target
5. Nomad places a Linux container on macOS — runtime failure

This failure is insidious because:
- It requires two independent events separated by an arbitrary time gap
- The runtime error does not point back to the placement decision
- There is no audit trail connecting the eligibility change to the failure

### Why Job-File Constraints Are Sufficient

The constraint block in a job file is:
- **Version-controlled**: Changes require a commit with an audit trail
- **Job-specific**: Applies only where needed, not cluster-wide
- **Evaluated at scheduling time**: Prevents incorrect placement before it happens
- **Survives everything**: Cluster restarts, node restarts, operator changes

### Belt-and-Suspenders

The recommended practice uses both mechanisms together:
- **Job-file constraint**: Prevents incorrect placement (safety net)
- **Node eligibility**: Prevents unnecessary scheduling evaluations (optimization)

Neither alone is wrong. Together, they provide defense in depth.

### Standing Convention

Three jobs established this pattern in Phase 21:

| Job | Type | Purpose |
|-----|------|---------|
| node-exporter | system | Host metrics collection (all Linux nodes) |
| observability-lifecycle | batch (periodic) | Data retention management |
| log-event-collector | batch (periodic) | Infrastructure event collection |

**Convention**: Any future Docker job with a Linux container image must carry
the macOS exclusion constraint. The convention is self-enforcing: omitting it
risks a runtime failure on the next rescheduling event.

---

## Consequences

### Positive

- **Immediate unblocking**: Removing `service {}` blocks restored four
  services within minutes of the incident
- **Durable placement safety**: Job-file constraints prevent Linux containers
  on macOS regardless of operational state changes
- **Git-traceable policy**: All placement decisions are version-controlled
- **Implicit audit discovery**: Forced rescheduling functioned as an
  unintentional cluster-wide constraint validation pass

### Negative

- **Services leave Consul catalog**: Four services on the affected node
  are no longer discoverable via Consul (acceptable — no consumers used
  Consul to find them)
- **Manual constraint maintenance**: Each new Linux Docker job requires
  the developer to add the exclusion constraint. No automated enforcement
  exists.
- **Split discovery model**: The cluster uses Consul on two nodes and
  Nomad-native on two nodes, increasing cognitive load

### Neutral

- If Consul is installed on the affected node in a future phase, `service {}`
  blocks can be re-added. The decision is reversible.

---

## Key Insight: Infrastructure State Durability Spectrum

Infrastructure state exists on a spectrum from ephemeral to durable:

```
Most ephemeral                                      Most durable
     |                                                   |
     v                                                   v
  Runtime     Node         Cluster        Job file     Git
  memory    eligibility    variables     constraint   history
             flags                       blocks
```

Placement policy is an architectural decision. Encoding it as ephemeral
operational state creates a time bomb: it works until someone resets
the state.

This principle applies beyond Nomad. Any system where "where does this
run?" is answered by mutable runtime state rather than declarative
configuration has the same class of drift vulnerability.

---

## Lessons Learned

### Forced Rescheduling as a Cluster Audit

Any event that forces a scheduler to re-evaluate placement (node restart,
container runtime restart, leader election) is effectively a cluster-wide
constraint validation pass. Configurations that were valid at initial
placement may not be valid when re-evaluated against current infrastructure.

This is analogous to the difference between `CREATE TABLE IF NOT EXISTS`
(evaluated once) and a foreign key constraint (evaluated on every write).

### Service Registration Without an Agent

The orchestrator's handling of orphaned service registration blocks is
nuanced:
- **At initial placement**: Registration attempt fails, job is rejected
- **At runtime**: Agent disappearing does not affect running workloads
- **At rescheduling**: Registration requirement is re-evaluated, failure
  surfaces

A service block can be "working" (allocation running) and "broken"
(service not registered) simultaneously. The breakage only becomes visible
when rescheduling forces re-evaluation.

---

**ADR-020 Status**: Accepted (2026-04-12)
