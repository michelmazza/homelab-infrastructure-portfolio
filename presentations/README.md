# Presentations

Professional slide decks documenting the infrastructure journey from foundation to AI/ML optimization.

---

## Infrastructure Overview (Phases 4-6)

### Foundation & Production Operations
**File**: infrastructure-overview.pdf  
**Slides**: 12 slides  
**Format**: PDF  
**Coverage**: Phases 4-6

Visual presentation of the infrastructure foundation:
- **Phase 4**: Production Operations (15+ services, NFS storage)
- **Phase 5**: Observability Stack (Prometheus, Grafana, alerting)
- **Phase 6**: AI/ML Foundation (Ollama deployment)
- **Key Story**: The Hybrid Decision (CPU vs GPU vs Hybrid)
- **Architecture**: Phase 6 system diagram with all components

**Highlights**:
- Nomad cluster evolution
- Service mesh with Consul
- Observability maturity
- Hybrid Ollama architecture (24× GPU speedup)
- Production metrics and achievements

**Use for**: General technical overview, infrastructure discussions

---

## Phase 7: RAG Platform Foundation

### Production Knowledge System
**File**: phase-7-rag-platform.pdf  
**Slides**: 15 slides  
**Format**: PDF  
**Coverage**: Phase 7 (Advanced RAG Platform)

Complete RAG platform creation story:
- **Architecture**: RAG system diagram with 5-layer pipeline
- **Pipeline Flow**: Detailed query flow (cache → retrieval → LLM)
- **Production Metrics**: 611 chunks, 85 docs, 200× cache speedup
- **Engineering Discovery**: "The Retrieval Ceiling" (systematic debugging)
- **Foundation**: Sets up Phase 8 optimization story

**Highlights**:
- 5-layer swim lane architecture diagram
- Cache hit (10ms) vs full retrieval (2.5s) paths
- Conversation persistence with SQLite WAL mode
- JSON embeddings strategy (15MB, 768 dimensions)
- Systematic debugging methodology

**Key Achievement**: 15 docs → 85 docs (5.7× scale), production-ready platform

**Use for**: RAG platform discussions, Phase 8 context, engineering maturity

---

## Phase 8: Retrieval Quality Optimization

### The 90% Accuracy Journey
**File**: phase-8-optimization-journey.pdf  
**Slides**: 13 slides  
**Format**: PDF  
**Coverage**: Phase 8 (Retrieval Optimization)

The breakthrough optimization story:
- **The Challenge**: 0% accuracy discovery
- **The Approach**: Three-layer systematic strategy
- **The Breakthrough**: Session 2 (+55pp from document boosting)
- **The Results**: 90% accuracy achieved
- **Key Techniques**: Hybrid search, document boosting, topic authority
- **Lessons Learned**: Data-driven engineering principles

**Highlights**:
- Session-by-session progress visualization
- Three-layer boosting architecture
- Results tables and category breakdowns
- Time investment analysis (7.5 hours total)
- Professional quality for interviews

**Key Achievement**: 0% → 90% accuracy, exceeded 60-70% target by 20-30pp

**Use for**: Optimization discussions, systematic engineering, interview storytelling

---

## Story Arc (Complete)

**Phase Progression**:
1. **Infrastructure Overview** (Phases 4-6): Foundation and production operations
2. **Phase 7**: Build the RAG platform (production-ready system)
3. **Phase 8**: Optimize systematically (0% → 90% accuracy)

**Interview Narrative**: *"We built solid infrastructure, created a production RAG platform, then optimized it systematically to achieve 90% accuracy - exceeding targets through data-driven engineering."*

---

## Presentation Philosophy

All presentations follow core engineering principles:
- **Visual Clarity**: Architecture diagrams at multiple zoom levels
- **Data-Driven**: Metrics and results prominently featured
- **Systematic Approach**: Methodical problem-solving showcased
- **Engineering Honesty**: Problems positioned as discoveries
- **Professional Quality**: Interview and presentation-ready

---

## Coverage Summary

| Presentation | Phases | Slides | Key Topics |
|--------------|--------|--------|------------|
| **Infrastructure Overview** | 4-6 | 12 | Nomad, Consul, Observability, Hybrid Ollama |
| **Phase 7** | 7 | 15 | RAG platform, 5-layer architecture, retrieval ceiling |
| **Phase 8** | 8 | 13 | 0% → 90% optimization, hybrid search, boosting |
| **Total** | **4-8** | **40** | **Complete technical journey** |

---

## Coming Soon

- **Phase 9**: Agentic RAG with MCP Tools (when Phase 9 completes)

---

**Status**: 3 presentations complete, covering Phases 4-8 with full visual storytelling ✅
