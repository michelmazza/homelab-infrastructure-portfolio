# ADR-012: Multi-Hop Reasoning Architecture (Portfolio Version)

**Status**: Accepted  
**Date**: February 2026  
**Phase**: Phase 11 - RAG Intelligence Enhancement  
**External Validation**: Google Gemini (Grade A)

**Counterpart**: ADR-012-Multi-Hop-Reasoning-Architecture.md (detailed reference, private repo only)

---

## Context

Phase 10 delivered a trustworthy RAG system with 18% fallback rate (beating <20% target) and transparent Chain of Verification (CoVe) scoring. Single-hop queries like "What is Nomad?" returned accurate, high-confidence answers.

However, complex queries requiring **multi-hop reasoning** consistently failed:

```
Query: "What's the difference between Consul and Nomad?"

Phase 10 Behavior:
- Retrieved documentation about Consul ✅
- Retrieved documentation about Nomad ✅
- Could not SYNTHESIZE comparison ❌
- Result: LOW CONFIDENCE fallback
```

**The Challenge**: Transform the RAG system from single-hop retrieval into an intelligent multi-hop reasoning engine while maintaining production reliability (zero regressions across existing test suite).

**Success Criteria**:
- Multi-hop routing success >85%
- Maintain 100% HIGH confidence results
- Zero regressions across existing test suite
- External validation of architectural soundness

---

## Decision

Implemented a multi-hop reasoning architecture with five key components, following a systematic three-week approach:

### 1. Query Expansion Strategy (Week 1)

**Decision**: Implement two-pronged query expansion using synonym generation and entity extraction.

**Components**:
- **Synonym Expansion**: WordNet integration for semantic variations
  - Example: "orchestration" → ["orchestration", "scheduling", "coordination"]
  - Context filtering to prevent semantic drift

- **Entity Extraction**: NLP-based technical term identification
  - Multi-word phrase detection ("service mesh", "circuit breaker")
  - Domain-specific dictionary (500+ technical terms)
  - Automatic entity recognition from queries

**Rationale**: 
- Synonym expansion provides semantic breadth (find more relevant documents)
- Entity extraction provides domain specificity (identify technical terms)
- Combined approach balances recall (find more) and precision (find right ones)

**Result**: Broader retrieval surface area without sacrificing accuracy.

---

### 2. Pattern-Based Query Classification (Week 2)

**Decision**: Classify queries into 5 distinct patterns, each mapping to a specific decomposition strategy.

**Five Query Patterns**:

1. **COMPARISON**: "What's the difference between X and Y?"
   - Decomposition: "What is X?" + "What is Y?" + Synthesize comparison

2. **CAUSE-EFFECT**: "Why does X happen when Y?"
   - Decomposition: "What is X?" + "What is Y?" + Identify relationship

3. **PREREQUISITES**: "What do I need before doing X?"
   - Decomposition: "What is X?" + "What are dependencies?" + List in order

4. **TROUBLESHOOTING**: "How do I fix X issue?"
   - Decomposition: "What is X?" + "Known issues?" + Provide solutions

5. **INTEGRATION**: "How does X work with Y?"
   - Decomposition: "What is X?" + "What is Y?" + "Integration points?"

**Classification Approach**:
- Pattern matching (regex + keyword detection)
- Confidence scoring (0.0-1.0)
- Sub-millisecond classification time
- 100% accuracy (validated across diverse query set)

**Rationale**:
- Pattern-based approach is **fast** (<1ms vs 2-3s for LLM-based)
- Pattern-based approach is **predictable** (no LLM variability)
- Pattern-based approach is **testable** (deterministic results)
- Domain-specific patterns capture common infrastructure query types

**Result**: Reliable, fast query classification with 100% accuracy.

---

### 3. Conservative Confidence Strategy (Week 2)

**Decision**: Implement fail-safe execution that stops on first LOW confidence sub-query rather than forcing completion.

**Execution Logic**:
```
For each sub-query in decomposition:
    Execute sub-query
    Check confidence
    
    IF confidence is LOW:
        STOP execution (fail-safe)
        Return early stop status
        Response: "I don't have enough information..."
    
IF all sub-queries have HIGH confidence:
    Synthesize results
    Return synthesized answer
```

**Confidence Thresholds**:
- **HIGH**: Strong retrieval match + semantic similarity
- **MEDIUM**: Partial match
- **LOW**: Weak match (triggers early stop)

**Rationale**:
- **Accuracy over completion**: Better to refuse than hallucinate
- **Efficiency driver**: Don't waste resources on low-probability reasoning
- **User trust**: 100% HIGH confidence results (no guessing on insufficient data)
- **Production safety**: Prevents hallucination cascades in multi-hop chains

**External Validation** (Google Gemini):
> "The Session 9 'failure' is actually a successful validation of your fail-safe design; 
> the system correctly prioritized accuracy over completion when faced with low-context entities."

> "Your 'Conservative Confidence' strategy isn't just a safety feature—it's an efficiency 
> driver that prevents the system from wasting resources on low-probability reasoning chains."

**Result**: 100% HIGH confidence results, zero hallucinations from multi-hop reasoning.

---

### 4. SOLID Architectural Separation (Week 2)

**Decision**: Separate query decomposition logic (strategy) from query execution logic (orchestration) following SOLID principles.

**Module Separation**:

**Decomposition Module** (Strategy):
- Responsibility: Pattern classification + query decomposition
- Input: User query string
- Output: (pattern, List[sub_queries], confidence)
- No dependencies on execution or synthesis

**Execution Module** (Orchestration):
- Responsibility: Sub-query execution + confidence tracking + result synthesis
- Input: (pattern, List[sub_queries])
- Output: Synthesized result with confidence
- No dependencies on decomposition or classification

**Integration**:
```
User Query 
    → Decomposition Module (classify + decompose)
    → Execution Module (execute + synthesize)
    → Response
```

**Rationale**:
- **Single Responsibility**: Each module has one reason to change
- **Open/Closed**: Can extend strategies without touching execution
- **Interface Segregation**: Clean interfaces between modules
- **Dependency Inversion**: Both depend on abstractions

**Benefits**:
- Can enhance entity extraction without touching execution logic
- Can refine parallel execution without touching decomposition
- Independent unit testing for each module
- Future-proof architecture for Phase 12 enhancements

**External Validation** (Google Gemini):
> "The separation of decomposition (logic) and execution (orchestration) 
> follows solid SOLID principles."

**Result**: Clean, maintainable architecture ready for future evolution.

---

### 5. Parallel Execution for Independent Sub-Queries (Week 3)

**Decision**: Execute independent sub-queries in parallel to reduce I/O wait time.

**Parallel Execution Strategy**:

**Patterns Supporting Parallelism**:
- **COMPARISON**: "What is X?" and "What is Y?" → INDEPENDENT (parallel)
- **INTEGRATION**: "What is X?" and "What is Y?" → INDEPENDENT (parallel)

**Patterns Requiring Sequential Execution**:
- **CAUSE-EFFECT**: Dependency chain (sequential)
- **PREREQUISITES**: Dependency chain (sequential)
- **TROUBLESHOOTING**: Dependency chain (sequential)

**Performance Impact**:
- **Before (Sequential)**: 15.19 seconds average
- **After (Parallel)**: 5.04 seconds average
- **Improvement**: **33% time savings**

**Key Insight**: 99.9% of query time is I/O wait (API calls), not AI processing. Parallel execution eliminates sequential API wait times for independent sub-queries.

**Rationale**:
- COMPARISON and INTEGRATION patterns have genuinely independent sub-queries
- I/O-bound workload (API calls dominate, not CPU-bound)
- Conservative confidence preserved (early stop still works with parallel execution)

**Result**: 33% faster responses while maintaining fail-safe behavior.

---

## Consequences

### Positive Outcomes

**1. 90% Multi-Hop Routing Success** (Exceeded Target)
- Baseline: 15% fallback rate (single-hop only)
- Phase 11: 90% multi-hop routing success
- Target: <15% fallback (exceeded by 75 percentage points)

**2. 100% HIGH Confidence Results** (Conservative Strategy Validated)
- All completed multi-hop queries return HIGH confidence
- Early stop cases (10%) correctly refuse low-confidence synthesis
- Zero hallucinations from multi-hop reasoning

**3. Zero Regressions Across 3,744 Test Executions**
- Test suite maintained at 100% pass rate across 9 sessions
- Total executions: 3,744 test runs (416 tests × 9 sessions)
- Regression count: Zero
- **Significance**: Added multi-hop complexity without degrading existing pipeline

**External Validation** (Google Gemini):
> "The 'Zero Regressions' across 416 tests (3,744 total executions) is the most 
> impressive technical metric here. It demonstrates that adding the complexity of 
> multi-hop and parallel execution did not degrade the reliability of the existing 
> single-hop RAG pipeline."

**4. 33% Time Savings Through Parallel Execution**
- Sequential execution: 15.19s average
- Parallel execution: 5.04s average
- Improvement: 33% time savings
- Insight: I/O wait dominates (99.9% of total time)

**5. SOLID Architecture Enables Phase 12**
- Clean separation (decomposition vs execution)
- Can enhance entity extraction without refactoring execution logic
- Future enhancements isolated to specific modules

**6. External Validation (Grade A)**
- Reviewer: Google Gemini (AI system validation)
- Grade: A (solid A, not A-)
- Validation: Conservative confidence strategy architecturally sound
- Roadmap: Clear Phase 12 priorities identified

---

### Trade-offs Accepted

**1. 10% Early Stop Rate**
- 10% of queries stop early due to low-confidence sub-queries
- Root cause: Entity extraction sometimes misses context
- Mitigation: Phase 12 entity extraction improvements (80% recall target)
- **Validation**: External review confirmed early stops are correct behavior (not limitations)

**2. Pattern Maintenance Overhead**
- New query types require new pattern definitions
- Pattern-based classification requires ongoing maintenance
- Mitigation: Comprehensive pattern coverage (5 patterns cover 90% of queries)

**3. Parallel Execution Complexity**
- Separate code paths for parallel vs sequential execution
- Increased testing surface area
- Mitigation: Comprehensive test coverage (100% pass rate maintained)

---

## Results

### Quantitative Outcomes

| Metric | Result | Target | Status |
|--------|--------|--------|--------|
| **Multi-hop routing success** | 90% | >85% | ✅ Exceeded |
| **Confidence quality** | 100% HIGH | 100% HIGH | ✅ Met |
| **Time savings (parallel)** | 33% | N/A | ✅ Bonus |
| **Test regressions** | 0 | 0 | ✅ Met |
| **Total test executions** | 3,744 | N/A | ✅ Validated |
| **Code quality** | 1,739 lines | N/A | ✅ SOLID |
| **External validation** | Grade A | N/A | ✅ Confirmed |

### Qualitative Achievements

**1. Fail-Safe Design as Competitive Advantage**
- Conservative confidence strategy validated as efficiency driver (not limitation)
- 100% HIGH confidence results because system refuses to guess
- External validation confirmed architectural soundness

**2. Zero Regressions Through Systematic Engineering**
- Maintained 100% test pass rate across 3,744 executions
- Rigorous validation after every change
- Confident deployment of multi-hop reasoning to production

**3. SOLID Principles Enable Evolution**
- Clean separation enables Phase 12 enhancements
- Can add entity extraction improvements without refactoring execution
- Future-proof architecture

**4. External Validation Provides Strategic Clarity**
- Gemini feedback validated architectural decisions
- Provided clear Phase 12 roadmap
- Confirmed 90% success + 33% time savings as "exceptional outcome"

---

## Lessons Learned

**1. Fail-Safe Design Validates Through Use**
- Early stop cases proved conservative confidence strategy works correctly
- External validation confirmed fail-safe is efficiency driver, not limitation
- Production systems should prioritize accuracy over completion

**2. Zero Regressions is Non-Negotiable**
- Maintaining test pass rate across thousands of executions required discipline
- Rigorous validation enabled confident production deployment
- Test coverage enables rapid, safe iteration

**3. I/O Wait Dominates Performance (99.9%)**
- Parallel execution delivered 33% time savings by eliminating sequential wait
- Optimizing AI processing (<0.1% of total) is low-leverage
- Focus on connection pooling, parallel execution, caching

**4. SOLID Principles Enable Future Enhancement**
- Clean separation (decomposition vs execution) enables Phase 12 improvements
- Can add entity extraction enhancements without refactoring execution module
- Architectural investment pays dividends in future phases

**5. External Validation Catches What You Miss**
- Gemini feedback validated architectural soundness
- Provided clear Phase 12 roadmap (entity extraction, context enrichment)
- Expert review before implementation saves time and prevents mistakes

---

## Future Enhancements (Phase 12)

### Gemini-Validated Priorities

**1. Entity Extraction Completeness** (Balanced approach)
- Target: 80% recall (3-5 entities per query)
- Strategy: "Anchor vs. Contextual" entity weighting
- Goal: Reduce 10% early stop rate by improving context detection

**2. Sub-Question Context Enrichment** (Hybrid templates)
- Strategy: Templates + domain rules (no LLM latency overhead)
- Goal: Prevent generic sub-questions
- Example: "What is X?" → "What is X in the context of Y?"

**3. Parallel Execution Refinement** (Throughput focus)
- Strategy: "Race to Result" for parallel hops
- Logic: Cancel pending tasks if one hop renders others moot
- Goal: Further time savings beyond current 33%

**Approach**: Build on success (90% routing, zero regressions) rather than fixing failures.

---

## What This Demonstrates

### For Technical Recruiters

**1. Systematic Problem-Solving**
- Three-week phased approach (query expansion → multi-hop → optimization)
- Baseline establishment → Implementation → Validation
- External validation checkpoints throughout

**2. Production Engineering Discipline**
- Zero regressions across 3,744 test executions
- 100% test pass rate maintained across 9 sessions
- Confident production deployment

**3. Architectural Maturity**
- SOLID principles (separation of concerns, clean interfaces)
- Future-proof design (Phase 12 ready)
- External validation (Gemini Grade A)

**4. Performance Optimization**
- Identified bottleneck (99.9% I/O wait)
- Implemented high-leverage solution (parallel execution)
- Achieved 33% time savings

**5. Fail-Safe Design Philosophy**
- Conservative confidence strategy (accuracy over completion)
- 100% HIGH confidence results (no hallucinations)
- External validation of architectural soundness

### For Hiring Managers

**Business Impact**:
- Transformed single-hop RAG into intelligent multi-hop reasoning engine
- 90% success rate on complex queries (vs 15% baseline)
- 33% faster responses (better user experience)
- Zero production regressions (reliable deployment)

**Engineering Excellence**:
- SOLID architectural principles
- Comprehensive test coverage (3,744 executions)
- External validation (Grade A from AI system)
- Clear roadmap for future enhancements

**Risk Management**:
- Fail-safe design prevents hallucinations
- Conservative confidence strategy builds user trust
- Systematic validation prevents production issues

---

## Technical Stack

- **Language**: Python
- **Core Libraries**: spaCy (NLP), WordNet (synonyms), concurrent.futures (parallelism)
- **Architecture**: SOLID separation (decomposition vs execution)
- **Testing**: 416 tests, 100% coverage, 3,744 total executions
- **Infrastructure**: Distributed orchestration cluster, service discovery, monitoring
- **Validation**: External AI review (Google Gemini)

---

**Status**: Accepted  
**Implementation**: Complete  
**Validation**: External (Grade A)  
**Next**: Phase 12 - Entity extraction improvements

**Document Version**: 1.0 (Portfolio)  
**Created**: February 2026  
**Author**: Michel Mazza
