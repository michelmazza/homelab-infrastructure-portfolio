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

### Innovation & AI/ML (Phases 6-8)
**AI/ML Foundation** → Local LLM deployment with Ollama  
**Advanced RAG Platform** → Production knowledge system with Streamlit UI  
**Retrieval Optimization** → 0% to 90% accuracy breakthrough ⭐  

### Evolution (Phase 9 - In Progress)
**Agentic RAG** → MCP tools integration for infrastructure automation  

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

**The Breakthrough**  
Session 2 diagnostic tool revealed ADR-001 was ranked #8 when it should be #1.  
**The Gap**: ADR-001 scored 0.4513 vs top-ranked document's 0.4990 - just 0.0477 points behind.  
Solution: Calculated 1.5× boost factor from this gap (data-driven, not guessed).  
Result: **20% → 75% accuracy in one 45-minute session** (+55pp breakthrough!)

Session 6 added topic authority: **75% → 90%** 🎉

**The Results**

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Overall Accuracy | 0% | 90% | +90pp 🏆 |
| Factual Queries | 0% | 80% | +80pp |
| Architectural | 0% | 100% | +100pp |
| Comparative | 60% | 100% | +40pp |
| How-To | 100% | 100% | Maintained |

**Time Investment**: 8 sessions, ~7.5 hours  
**Target**: 60-70% accuracy  
**Achieved**: 90% accuracy (exceeded by 20-30pp)  
**Performance**: ~25ms search latency (excellent)

**Key Techniques**
1. **Document Type Boosting**: ADRs=1.5×, Reports=1.3×, Sessions=0.8×
2. **Topic Authority**: Title-based matching for precise queries
3. **Hybrid Search**: BM25 (keywords) + Vector (semantics)

**Lessons Learned**
- ✓ Measure first, optimize second (test suite was essential)
- ✓ Data-driven decisions beat guessing (1.5× calculated from gap)
- ✓ Diagnosis beats complexity (diagnostic tool revealed exact fix)
- ✓ Know when to stop (90% exceeded target, declared victory)
- ✓ External validation catches bugs (Gemini feedback was valuable)

📖 **[Read the Full Phase 8 Story](journey/phase-8-optimization.md)**  
📊 **[View Detailed Metrics & Analysis](results/phase-8-metrics.md)**  
🎯 **[Phase 8 Presentation PDF](presentations/phase-8-optimization-journey.pdf)** (13 slides)  
🏛️ **[Architecture Decision (ADR-009)](architecture/decisions/ADR-009-retrieval-optimization.md)**

**Complete Story with Presentations**:  
📊 [Infrastructure Overview](presentations/infrastructure-overview.pdf) (Phases 4-6, 12 slides)  
📊 [Phase 7: RAG Platform](presentations/phase-7-rag-platform.pdf) (Foundation, 15 slides)  
📊 [Phase 8: Optimization](presentations/phase-8-optimization-journey.pdf) (90% Journey, 13 slides)

---

## What You'll Find Here

### 📖 [Journey](journey/)
Phase-by-phase narratives showing the evolution from empty cluster to production AI/ML platform. Problems faced, decisions made, results achieved.

**Available Now**:
- [Phase 8: 0% to 90% Optimization](journey/phase-8-optimization.md) - The breakthrough story

### 🏛️ [Architecture](architecture/)
Major architecture decisions (ADRs) documenting choices, trade-offs, and alternatives considered. Plus diagrams showing system topology and data flows.

**Available Now**:
- [ADR-001: Orchestration Platform](architecture/decisions/ADR-001-orchestration-platform.md) - Nomad vs K3s vs Swarm
- [ADR-008: Advanced RAG Platform](architecture/decisions/ADR-008-advanced-rag-platform.md) - Phase 7 foundation
- [ADR-009: Retrieval Optimization](architecture/decisions/ADR-009-retrieval-optimization.md) - 90% accuracy achieved

### 🎯 [Results](results/)
Metrics, improvements, and measurable outcomes. Before/after data, performance benchmarks, and achievement summaries.

**Available Now**:
- [Phase 8 Detailed Metrics](results/phase-8-metrics.md) - Complete performance analysis

### 📊 [Presentations](presentations/)
Technical presentations explaining complex work for different audiences. Slide decks telling the story visually.

**Available Now** (40 slides total):
- [Infrastructure Overview](presentations/infrastructure-overview.pdf) - Phases 4-6 foundation (12 slides)
- [Phase 7: RAG Platform](presentations/phase-7-rag-platform.pdf) - Production system (15 slides)
- [Phase 8: 90% Accuracy Journey](presentations/phase-8-optimization-journey.pdf) - Optimization story (13 slides)

### 📸 [Screenshots](assets/screenshots/)
Visual evidence of production infrastructure, observability, and AI/ML systems.

**Available Now** (10 screenshots):
- **Observability Stack** (Grafana dashboards, Prometheus targets, host metrics)
- **Infrastructure Services** (Nomad jobs, Consul services, Traefik routing)
- **AI/ML Platform** (Ollama monitoring - CPU + GPU hybrid deployment)
- **RAG User Interface** (Streamlit UI - welcome screen + query workflow with 90% accuracy in action)

**Coverage**: Complete system visibility from infrastructure through observability to user-facing AI/ML applications

### 🧠 [Methodology](methodology/)
How I approach engineering: systematic validation, data-driven decisions, ADR framework, session-based development, continuous improvement.

**Available Now**:
- Complete methodology documentation (6 core practices)
- Phase 8 case study demonstrating methodology in action
- Interview application guidance

---

## Key Achievements

**Infrastructure Orchestration**
- ✅ 3-node Nomad cluster (lab01, lab02, lab03)
- ✅ 15+ containerized services orchestrated
- ✅ Consul service mesh with health checks
- ✅ Traefik reverse proxy with automatic routing
- ✅ NFS distributed storage (10TB Synology NAS)

**Observability & SRE**
- ✅ Prometheus metrics collection and alerting
- ✅ 6 Grafana dashboards for comprehensive monitoring
- ✅ 5 production alert rules with Pushover notifications
- ✅ Loki log aggregation
- ✅ 99.9%+ uptime achieved

**AI/ML Engineering**
- ✅ Advanced RAG platform with 611 embedded chunks
- ✅ 90% search accuracy (0% → 90% optimization)
- ✅ Hybrid search (BM25 + vector embeddings)
- ✅ ~25ms search latency (production-grade performance)
- ✅ Ollama local LLM deployment
- ✅ MCP tools integration (Phase 9 in progress)

**Documentation & Process**
- ✅ 10+ comprehensive ADRs documenting major decisions
- ✅ 30+ session notes capturing development journey
- ✅ Systematic validation frameworks and test suites
- ✅ External feedback integration (AI-assisted reviews)
- ✅ Git-based version control with meaningful commits

---

## Technologies

### Infrastructure & Orchestration
**HashiCorp Nomad** • **Consul** • **Traefik** • **Docker** • **Proxmox** • **NFS**

### Observability & Monitoring
**Prometheus** • **Grafana** • **Loki** • **Pushover** • **AlertManager**

### AI/ML Platform
**Ollama** • **Sentence Transformers (BGE-M3)** • **BM25 (Custom Tokenizer)** •  
**Hybrid Search (BM25 + Vector)** • **Document Boosting** • **MCP Tools** • **Streamlit**

### Data & Storage
**SQLite (WAL Mode)** • **JSON Embeddings** • **Vector Embeddings** • **Multi-Tier Caching**

### Development & Process
**Python** • **Git** • **Markdown** • **YAML** • **ADRs** • **Session Notes**

---

## Current Status

**Phase 9: Agentic RAG with MCP Tools** (In Progress)  
Building an intelligent agent that integrates with infrastructure using Model Context Protocol. The RAG system becomes a co-pilot that can query live infrastructure state, diagnose issues, and provide contextual guidance.

**Week 1 Complete**: 5 MCP tools operational (Prometheus, Nomad, Consul APIs)  
**Next**: Intent classification, multi-step reasoning, production integration

---

## Engineering Principles

**Systematic Over Ad-Hoc**  
Every phase planned, every session documented, every decision captured in ADRs.

**Measure Before Optimizing**  
Phase 8 started with comprehensive test suite. Can't improve what you can't measure.

**Data-Driven Decisions**  
Document boost factor wasn't guessed (1.5×) - it was calculated from gap analysis.

**External Validation**  
Sought AI feedback at key milestones. Caught bugs, refined approaches, accelerated learning.

**Know When to Stop**  
Achieved 90% accuracy, target was 60-70%. Declared victory. Shipping beats perfection.

**Document Everything**  
Future-you and future-colleagues deserve context. ADRs, session notes, lessons learned.

---

## Why This Portfolio Matters

### For Recruiters & Hiring Managers
- **Real Production Work**: Not tutorials or toy projects
- **Measurable Results**: 0% → 90% accuracy, ~7.5 hours, systematic approach
- **Professional Process**: ADRs, validation frameworks, documentation
- **Modern Tech Stack**: Nomad, Consul, Prometheus, RAG, AI/ML
- **Continuous Learning**: 9 phases, each building on previous

### For Technical Peers
- **Architecture Decisions**: Detailed ADRs showing trade-off analysis
- **Systematic Methodology**: Validation frameworks, test suites, data-driven optimization
- **Lessons Learned**: What worked, what didn't, why
- **Real Constraints**: HomeLab limitations forcing creative solutions
- **Ongoing Evolution**: Phase 9 in progress, continuous improvement

### For Me
- **Portfolio of Work**: Tangible proof of capabilities
- **Learning Journey**: Documented evolution from Phase 1 to 9
- **Reference Material**: ADRs and session notes for future decisions
- **Professional Story**: Narrative for interviews and opportunities

---

## Connect

🔗 **LinkedIn**: [linkedin.com/in/michelmazza](https://www.linkedin.com/in/michelmazza/)  
📍 **Location**: Montreal, QC, Canada  

---

## About Me

I build production systems with systematic engineering practices. Every architecture decision documented in ADRs. Every optimization validated with data. Every lesson captured and applied to future phases.

**Current Focus**: Phase 9 - Agentic RAG with infrastructure integration

**Recent Achievement**: Phase 8 - 0% to 90% search accuracy in 7.5 hours

**Approach**: Systematic, data-driven, documented, continuously improving

---

## Contributing

This is a **personal portfolio** documenting my engineering journey. While I don't accept pull requests, I welcome feedback via [Issues](https://github.com/michelmazza/homelab-infrastructure-portfolio/issues).

Found a typo or broken link? Please open an issue!

Interested in the methodology? Feel free to fork and adapt to your own projects.

**License**: This work is licensed under [CC BY 4.0](LICENSE) - you're free to use and adapt with attribution.

---

---

## Portfolio Information

**Last Updated**: January 12, 2026  
**Portfolio Version**: 1.0 (Public Launch)  
**License**: [CC BY 4.0](LICENSE) - Free to use with attribution  

**Current Status**: 
- ✅ Phase 8 complete (90% RAG accuracy achieved)
- 🚧 Phase 9 in progress (MCP tools integration)
- 📋 Continuous documentation and methodology refinement

**Next Major Update**: Upon Phase 9 completion (MCP tools integration with live infrastructure APIs)

**Feedback**: Found an error or have suggestions? [Open an issue](https://github.com/michelmazza/homelab-infrastructure-portfolio/issues)

---

**Built with systematic engineering practices. Every decision measured. Every result validated.**

**Michel Mazza** | DevOps Engineer | AI/ML Systems Builder  
📍 Montreal, QC | 🔗 [LinkedIn](https://www.linkedin.com/in/michelmazza/) 
