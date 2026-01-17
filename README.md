# Infrastructure Engineering Portfolio

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

## Systematic DevOps & AI/ML Engineering

**Michel Mazza** | DevOps Engineer | AI/ML Systems Builder
📍 Montreal, QC | 🔗 [LinkedIn](https://www.linkedin.com/in/michelmazza/)

---

## About This Portfolio

This portfolio documents my journey building production-grade infrastructure and AI/ML systems from first principles. Every decision measured, every result validated, every lesson documented.

**Not tutorials. Not toy projects. Real production systems.**

---

## The Journey: 9 Phases of Evolution

### Foundation (Phases 1-3)
**Orchestration Platform** → Built 3-node HashiCorp Nomad cluster
**Service Discovery** → Implemented Consul for dynamic service mesh
**Reverse Proxy** → Deployed Traefik for intelligent routing

### Operational Maturity (Phases 4-5)
**Production Operations** → 15+ services orchestrated, NFS storage integrated
**Observability Stack** → Prometheus + Grafana + Loki with mobile alerting

### Innovation & AI/ML (Phases 6-9)
**AI/ML Foundation** → Local LLM deployment with Ollama
**Advanced RAG Platform** → Production knowledge system with Streamlit UI
**Retrieval Optimization** → 0% to 90% accuracy breakthrough ⭐
**Agentic RAG** → Intelligent infrastructure co-pilot with Glass Box AI ⭐

### Phase 9 Complete: Agentic RAG with Glass Box AI ✅

**The Transformation**: From static documentation library to intelligent infrastructure co-pilot

**Three-Week Build**:
- **Week 1**: MCP Tools Foundation (5 operational tools with security guardrails)
- **Week 2**: Agentic Reasoning (98.2% intent accuracy, 100% multi-step success)
- **Week 3**: Glass Box AI (visual transparency with 3-tab interface) ⭐

**Key Capabilities**:
- 5 MCP tools operational (Prometheus, Nomad, Consul APIs)
- Intent classification with 98.2% accuracy (<1ms response time)
- Cross-reference capability: 100% accuracy (live metrics vs documented thresholds)
- Multi-step reasoning: 100% success rate on complex diagnostic workflows
- **Glass Box AI**: Complete visual transparency into AI reasoning process

**Innovation Highlight**: Glass Box AI provides three visualization tabs:
- 🧠 **Thought Trace**: Step-by-step reasoning with duration and JSON details
- 📊 **Timeline**: Gantt chart showing temporal execution flow
- ⚡ **Performance**: Dashboard identifying bottlenecks and speedups

**Results**:
- 98.7% overall agentic accuracy (157/159 test cases passing)
- Response time: 21-121ms (41-238× faster than 5000ms engineering goal)
- 190 total tests (100% passing)
- **Key Discovery**: Glass Box AI revealed 99.9% of time spent in MCP API calls (not AI reasoning)

**Engineering Principles**:
1. Foundation First (MCP tools before reasoning)
2. Security by Default (read-only, rate-limited, validated)
3. Intent is Key (98.2% routing accuracy enables downstream success)
4. Simple Wins (pattern matching > machine learning for speed)
5. Transparency Reveals (visualization directed optimization strategy)
6. MVP Iteration (3 weeks, 3 deliverables, low-risk progression)

---

## 🏆 Featured: The 90% Accuracy Breakthrough

### Phase 8: Retrieval Quality Optimization

**The Challenge**
Built a production RAG (Retrieval-Augmented Generation) system in Phase 7. Fast, beautiful UI, solid infrastructure. But when I created a test suite: **0% accuracy on factual queries.**

Query: "What is Nomad?"
Expected: ADR-001 (authoritative definition)
Got: Random session notes 😱

**The Approach** (Initial Plan):
Systematic three-layer optimization:
- **Layer 1**: Hybrid Search (BM25 + Vector) - ✅ Implemented
- **Layer 2**: Small-to-Big Retrieval - ❌ Abandoned (chunks already optimal size)
- **Layer 3**: Systematic Experiments - ❌ Replaced with data-driven boosting

**The Pivot**: After Session 2's breakthrough with document boosting (+55pp!), pivoted from planned complex retrieval to simpler authority-based ranking. Data showed chunks were already 1847 chars (not 186 as assumed), making Small-to-Big unnecessary.

**The Breakthrough** (Session 2):
Document type boosting: ADRs and core docs ranked higher. **+55 percentage points in one session.**

**The Results**:
- **90% accuracy achieved** (exceeded 60-70% target by 20-30pp)
- Systematic engineering: measure → hypothesize → test → iterate
- Data-driven decisions: abandoned planned complexity when data showed simpler path
- Time investment: 7.5 hours over 5 sessions

**Key Insight**: Data-driven engineering beats complex solutions. The simplest boost (document authority) delivered the breakthrough.

[Read the full Phase 8 journey →](journey/README.md#phase-8-retrieval-quality-optimization)

---

## 📊 Portfolio Statistics

- **Infrastructure**: 3-node Nomad cluster, Consul service mesh, Traefik reverse proxy
- **Services Orchestrated**: 15+ production services
- **Observability**: Prometheus + Grafana + Loki with mobile alerting via Pushover
- **AI/ML Platform**: Ollama (CPU + GPU hybrid), RAG with 611 chunks across 85 documents
- **RAG Accuracy**: 90% retrieval accuracy → 98.7% agentic accuracy
- **Test Coverage**: 190 automated tests (100% passing)
- **Documentation**: 1,145-line journey narrative, ADRs, architecture diagrams, session notes
- **Presentations**: 54 professional slides across 4 presentations (14 slides on Glass Box AI)

---

## 📁 Repository Structure

```
homelab-portfolio/
├── README.md                                # This file
├── journey/                                 # Narrative documentation
│   └── README.md                            # Complete journey (1,145 lines)
├── presentations/                           # Slide decks (54 slides total)
│   ├── infrastructure-overview.pdf          # Phases 4-6 (12 slides)
│   ├── phase-7-rag-platform.pdf             # Phase 7 (15 slides)
│   ├── phase-8-optimization-journey.pdf     # Phase 8 (13 slides)
│   └── Phase-9-Presentation-Agentic-RAG.pdf # Phase 9 (14 slides) ⭐
├── architecture/                            # Technical decisions
│   ├── decisions/                           # ADRs documenting key choices
│   └── diagrams/                            # System architecture visuals
├── assets/                                  # Screenshots and visuals
│   └── screenshots/                         # Glass Box AI interface screenshots ⭐
├── results/                                 # Metrics and analysis
│   ├── phase-8-metrics.md                   # Session-by-session breakdown
│   └── test-results/                        # Automated test outputs
└── methodology/                             # Engineering approach
    └── documentation-strategy.md
```

---

## 🎯 Quick Navigation

- **For Technical Depth**: [Journey Narrative](journey/README.md) - Complete story with technical details
- **For Visual Overview**: [Presentations](presentations/) - 54 slides covering Phases 4-9
- **For Engineering Decisions**: [Architecture Decisions](architecture/decisions/) - ADRs explaining key choices
- **For Metrics**: [Results](results/) - Quantified outcomes and test coverage
- **For Screenshots**: [Assets](assets/screenshots/) - Glass Box AI interface ⭐

---

## 🚀 Highlighted Innovations

### Glass Box AI (Phase 9) ⭐
**The Innovation**: Complete visual transparency into AI reasoning process

Traditional AI agents are black boxes - users see outputs but not reasoning. Glass Box AI exposes every step, tool call, and performance metric through three interactive tabs:

1. **Thought Trace**: Step-by-step reasoning with expandable JSON details
2. **Timeline**: Plotly Gantt chart showing temporal execution flow  
3. **Performance**: Dashboard with bottleneck identification and speedup visualization

**The Impact**: 
- Revealed 99.9% of execution time in MCP API calls (not AI reasoning)
- Directed optimization strategy toward external APIs (not AI logic)
- Built user trust through complete transparency
- Enabled data-driven engineering decisions

**The Differentiator**: Not just "what AI can do" but "showing HOW it does it"

[See Glass Box AI screenshots →](assets/screenshots/)

### The 90% Accuracy Breakthrough (Phase 8)
**The Challenge**: 0% accuracy on factual queries despite solid infrastructure

**The Approach**: Systematic three-layer optimization with data-driven pivots

**The Breakthrough**: Document type boosting (+55pp in Session 2)

**The Result**: 90% accuracy achieved, exceeding 60-70% target by 20-30pp

[Read the optimization journey →](journey/README.md#phase-8-retrieval-quality-optimization)

---

## 📈 Engineering Approach

**Systematic Methodology**:
- Measure first, optimize second
- Data-driven decision making
- Iterative improvement with measurable milestones
- Comprehensive documentation of decisions and outcomes
- Test coverage for validation

**Key Principles**:
- Foundation before intelligence (build reliable base first)
- Security by default (not retrofitted)
- Simple wins (complexity when justified by data)
- Transparency reveals truth (visualization guides optimization)
- MVP iteration (deliver value incrementally)

---

## 📞 Contact & Links

- **LinkedIn**: [linkedin.com/in/michelmazza/](https://www.linkedin.com/in/michelmazza/)
- **Location**: Montreal, QC, Canada
- **Portfolio**: This repository documents real production systems and engineering approach

---

## 📝 License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

You are free to:
- **Share**: Copy and redistribute the material
- **Adapt**: Remix, transform, and build upon the material

Under the following terms:
- **Attribution**: Give appropriate credit and link to license

---

**Last Updated**: January 15, 2026
**Portfolio Version**: 1.1
**Status**: Phase 9 Complete ✅ | Glass Box AI Innovation ⭐
