# Architecture

Architecture decisions, system diagrams, and technology choices.

---

## Architecture Decision Records (ADRs)

### ADR-009: Phase 8 Retrieval Quality Optimization
**File**: [decisions/ADR-009-retrieval-optimization.md](decisions/ADR-009-retrieval-optimization.md)  
**Date**: January 5-6, 2026  
**Status**: ✅ Complete

Systematic optimization of RAG retrieval achieving 90% accuracy through three-layer boosting architecture: Hybrid Search (BM25 + Vector), Document Type Boosting, and Topic Authority.

**Key Decisions**:
- Hybrid search with α=0.3 ratio
- Document authority boosting (ADR=1.5×, Sessions=0.8×)
- Topic authority via title matching
- Rejected Small-to-Big retrieval (chunks already 1847 chars)

---

## Architecture Diagrams

**Coming Soon**:
- RAG Pipeline Architecture
- Three-Layer Boosting System
- Nomad Cluster Topology
- Observability Stack

---

## Technology Stack

Complete overview of technologies used across all phases.

**Coming Soon**: Comprehensive tech stack documentation

---

**Status**: Content being added incrementally
