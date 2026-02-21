# ADR-013: LLM-Guided Query Decomposition with Hybrid Validation

**Status**: Accepted ✅  
**Date**: 2026-02-20  
**Decision Owner**: Infrastructure Team  
**External Validation**: A+ grade

> **Note**: This is the portfolio version (executive summary). For comprehensive technical details, see the private repository version.

---

## Executive Summary

**Problem**: Pattern-based query decomposition achieved only 15% quality rating (GOOD), limiting the effectiveness of the RAG system's multi-step reasoning capabilities.

**Decision**: Implement hybrid LLM-guided decomposition combining semantic understanding with deterministic validation.

**Result**: 4.4× improvement (15% → 66.7% GOOD rate), 100% precision (zero poor-quality decompositions), production-ready latency (3.95s, 21% under target).

---

## Context

### The Challenge

The agentic RAG system had sophisticated retrieval and validation capabilities, but query decomposition was a critical weakness:

**Baseline Performance** (Pattern-based approach):
- GOOD rate: 15% (only 2 of 13 decompositions helpful)
- Precision: 50% (half added no value)
- Symptom: Generic sub-questions like "What is X?" instead of entity-rich, actionable queries

**Root Causes**:
1. **Rigid templates**: Fixed patterns couldn't adapt to query variations
2. **Context blindness**: Keyword matching missed user intent
3. **Entity issues**: Focused on nouns, missed relationships
4. **Low precision**: Conservative validation prevented worse results but couldn't improve quality

### Strategic Context

**Phase 13 Goal**: Improve pattern-based approach with better entity extraction
- Result: 30% decomposition rate (up from ~10%), but still 15% GOOD rate

**Phase 14 Pivot**: Replace pattern matching with LLM semantic understanding
- Hypothesis: LLMs can capture intent that patterns miss
- Risk: Hallucination, latency, non-determinism
- Mitigation: Hybrid architecture with validation layers

**Success Criteria**:
- GOOD rate: >40% (2.5× improvement minimum)
- Precision: >75% (reduce unhelpful decompositions)
- Zero regressions: System stability maintained
- Latency: <5s (production requirement)

---

## Decision

### Hybrid LLM-Guided Architecture

Implement a system combining LLM semantic understanding with deterministic validation:

**Core Components**:

1. **LLM Intent Classifier**
   - Classifies queries into 7 intent types
   - Local LLM inference (~0.5s latency)
   - Replaces keyword matching

2. **Intent Taxonomy** (7 types):
   - temporal: Evolution, timeline changes
   - aggregation: Synthesis, overview
   - conditional: If-then scenarios
   - comparison: Contrasting alternatives
   - process_flow: Sequences, workflows
   - integration_chain: Component interactions
   - diagnostic: Troubleshooting, root cause

3. **LLM Sub-Question Generator**
   - Generates sub-questions with pattern template guidance
   - Entity-aware prompting
   - Intent-specific strategies

4. **Three-Layer Validation**
   - Entity coverage (50% threshold)
   - Duplication detection (0.85 similarity)
   - Quality assessment (GOOD/FAIR/POOR)

5. **Conservative Threshold**
   - 0.5 confidence minimum
   - Prioritizes precision over recall

### Key Design Decisions

**D1: Hybrid vs Pure LLM**
- Chosen: Hybrid (LLM understanding + pattern structure)
- Rationale: Best of both worlds - semantic understanding with reliability
- Result: 4.4× improvement, 100% precision

**D2: 7 Intent Types**
- Covers 90%+ of observed query patterns
- Specific enough for tailored decomposition
- Scalable: No quality loss when expanding from 4 → 7 intents

**D3: Conservative 0.5 Threshold**
- Prioritizes 100% precision over higher decomposition rate
- Blocks ~20% of queries
- Phase 15 opportunity: Lower to 0.3-0.4 (unlock +10-15pp GOOD rate)

**D4: Entity-Aware LLM Prompts**
- Always provide extracted domain entities to LLM
- Dramatically improves sub-question specificity
- Example: "What is routing?" → "How does multi-hop routing work in query pipeline?"

**D5: Maintain Validation Rigor**
- Keep deterministic 3-layer validation
- Enhance quality assessment with LLM
- Prevents hallucination, ensures production quality

---

## Options Considered

### Option 1: Pure Pattern-Based (Status Quo)

**Pros**: Fast (<0.5s), deterministic, explainable  
**Cons**: 15% GOOD rate, 50% precision, rigid templates, context-blind

**Decision**: **Rejected** - Hit quality ceiling, can't scale to production

---

### Option 2: Pure LLM-Based (No Patterns)

**Pros**: Maximum semantic understanding, handles edge cases, simpler architecture  
**Cons**: Hallucination risk, no structural guarantees, non-deterministic, ~5-6s latency

**Decision**: **Rejected** - Too risky, no quality guarantees

---

### Option 3: Hybrid LLM + Pattern Architecture (Chosen)

**Pros**: 66.7% GOOD rate (4.4× improvement), 100% precision, semantic + structural, 3.95s latency  
**Cons**: More complex, higher latency than patterns, LLM inference cost

**Decision**: **Chosen** - Empirically validated, production-quality results

**Why**:
- Week 2: 50% GOOD (3.3× improvement) → Proceed
- Week 3: 66.7% GOOD (4.4× improvement) → Ship
- 100% precision maintained (zero poor decompositions)
- Production latency met (3.95s < 5s target)
- External validation: A+ grade

---

## Results

### Primary Metrics

| Metric | Baseline | Final | Improvement | Status |
|--------|----------|-------|-------------|--------|
| **GOOD Rate** | 15% | 66.7% | **4.4×** | ✅ Exceeded |
| **Precision** | 50% | 100% | **2.0×** | ✅ Perfect |
| **Intent Coverage** | 0 | 7 types | **+7** | ✅ Complete |
| **POOR Rate** | 31% | 0% | **-31pp** | ✅ Eliminated |
| **Avg Latency** | N/A | 3.95s | **21% under target** | ✅ Production |

### Quality Distribution Evolution

**Baseline** (Pattern-based):
- GOOD: 15%, FAIR: 54%, POOR: 31%

**Final** (LLM-guided):
- GOOD: 66.7%, FAIR: 33.3%, POOR: 0%

**Achievement**: Complete elimination of poor-quality decompositions

### Week-by-Week Progression

**Week 1**: Intent classification foundation
- 85% accuracy, 100% consistency
- 4 intent types validated

**Week 2**: LLM-guided generation (4 intents)
- 50% GOOD rate → **3.3× improvement**
- 100% precision (zero poor decompositions)
- **74% of total improvement delivered**

**Week 3**: 7-intent expansion
- 66.7% GOOD rate → **4.4× improvement**
- 100% precision maintained
- No quality loss from scaling

### Real-World Example

**Query**: "Compare system accuracy in Version 1 vs Version 2"

**Baseline (POOR)**:
- "What is Version 1?" (too generic)
- "What is Version 2?" (too generic)
- "What is accuracy?" (unhelpful)

**Final (GOOD)**:
- "What was system accuracy in Version 1?" (specific metric)
- "What accuracy did Version 2 achieve?" (specific metric)
- "What improvements were made between V1 and V2?" (delta analysis)

**Impact**: LLM captured comparison intent correctly, generated actionable sub-questions

---

## Consequences

### Positive Outcomes ✅

**1. Quality Transformation**
- 4.4× GOOD rate improvement
- 2× precision improvement
- 7 operational intent types
- Zero pattern fallbacks

**2. Semantic Understanding**
- LLM correctly classifies intent (comparison vs temporal)
- Entity-aware generation (specific vs generic questions)
- Handles query variations naturally

**3. Production Performance**
- 3.95s average latency (21% under 5s target)
- Zero new regressions (97.6% test pass rate)
- 100% precision maintained across 3 weeks

**4. Scalability Validated**
- 4 intents → 7 intents: No quality degradation
- Foundation for future expansion (ranking, summarization, exploration)

**5. External Validation**
- A+ grade from independent review
- Praised: Methodology, data-driven decisions, conservative thresholds
- Suggested: Threshold tuning, additional intents (now in backlog)

### Trade-offs ⚠️

**1. Latency Increase**
- Pattern-based: <0.5s
- LLM-guided: 3.95s (7-8× slower)
- Mitigation: Within production requirement, optimization opportunity exists

**2. Architectural Complexity**
- More components (LLM classifier + generator + 3-layer validation)
- More code paths (7 intent types × strategies)
- Mitigation: Comprehensive testing (454 tests), clear taxonomy

**3. Conservative Threshold Cost**
- Blocks ~20% of queries (0.5 confidence minimum)
- Opportunity: Phase 15 can lower to 0.3-0.4 (unlock +10-15pp GOOD rate)
- Rationale: Precision over recall (better to not decompose than decompose poorly)

### Rollback Plan

**Scenario 1: Latency Exceeds 5s**
- Action: Increase threshold to 0.6-0.7 (reduce LLM calls)
- Fallback: Pattern-based code preserved

**Scenario 2: Quality Degrades <60% GOOD**
- Action: Audit validation layers, adjust scoring
- Rollback: Revert to Phase 14 Week 3 baseline (66.7% proven stable)

**Scenario 3: Poor Decompositions Re-emerge**
- Action: Raise threshold temporarily, investigate validation gap
- Fix: Patch within 1 session

---

## Implementation Highlights

### LLM Intent Classification

Semantic understanding replaces keyword matching:

**Example Success**:
- Query: "How did feature X improve from V1 to V2?"
- Pattern: temporal (keyword "improve")
- LLM: **comparison** ✅ (contrasting approaches)
- Impact: Correct intent → better sub-questions

**Validation** (20 queries):
- Accuracy: 85% (17/20 correct)
- Consistency: 100% (deterministic)

### Entity-Aware Generation

Entity context dramatically improves quality:

**Without entities**: "What is routing?" (generic)  
**With entities**: "How does multi-hop routing work in query pipeline?" (specific)

**Impact**: Entity coverage 40% → 85%

### Three-Layer Validation

Prevents hallucination, ensures quality:

**Layer 1**: Entity coverage → 100% pass rate  
**Layer 2**: Duplication detection → 3% duplicate rate  
**Layer 3**: Quality assessment → 0% POOR rate

**Result**: 50% → 100% precision

### MVP Approach

Incremental validation at each step:

- Week 1: Test intent classification → 85% → Proceed
- Week 2: Test 4 intents → 50% GOOD (3.3×) → Proceed
- Week 3: Test 7 intents → 66.7% GOOD (4.4×) → Ship

**Pattern**: Fail fast, validate early, scale systematically

---

## Key Learnings

**1. Hybrid Architectures Win**
- LLM semantic understanding + pattern structure > either alone
- Don't replace deterministic logic - augment it

**2. MVP Approach Validated**
- 74% of improvement came from Week 2 (first good implementation)
- Refinements add incremental gains, breakthrough happens early

**3. Conservative Thresholds Prevent Degradation**
- 0.5 confidence → 100% precision
- Can tune later, can't recover from shipping poor quality

**4. Entity Context Essential**
- Improves LLM output from generic to specific
- Entity extraction isn't just for patterns - critical for LLM quality

**5. Validation Layers Non-Negotiable**
- LLMs powerful but not infallible
- Deterministic validation prevents hallucination

**6. Measure Twice, Cut Once**
- Every decision data-driven, empirically validated
- No assumptions, only measurements

**7. External Validation Catches Blind Spots**
- A+ grade provided enhancement suggestions
- Now in Phase 15 backlog (threshold tuning, additional intents)

**8. Production Quality Requires Discipline**
- Zero regressions, 100% precision maintained across 3 weeks
- Result of systematic testing, validation, conservative engineering

---

## Future Work

### High Priority: Confidence Threshold Tuning

**Opportunity**: Lower from 0.5 → 0.3-0.4  
**Hypothesis**: Unlock +10-15pp GOOD rate  
**Risk**: Low (validation prevents degradation)  
**Effort**: 1-2 sessions

### Medium Priority: Additional Intent Types

**Candidates**: Ranking, summarization, exploration  
**Foundation**: 7-intent expansion showed no quality loss  
**Approach**: Incremental (1-2 per phase)

### Medium Priority: Latency Optimization

**Current**: 3.95s  
**Target**: 2.5-3.0s (50% margin)  
**Opportunities**: Prompt caching (-40%), approximate matching (-50%)  
**Effort**: 2-3 sessions

### Low Priority: Advanced Retrieval

**Focus**: Algorithm tuning, evaluation frameworks  
**Foundation**: High-quality decomposition unlocks retrieval improvements

---

## Monitoring & Success Metrics

**Quality Metrics**:
- GOOD rate: Target >60%, baseline 66.7%
- Precision: Target 100%
- Intent coverage: 7/7 operational

**Performance Metrics**:
- Average latency: Target <5s, baseline 3.95s
- P95 latency: Target <7s

**Regression Metrics**:
- Test pass rate: Target >97%, baseline 97.6%
- New regressions: Target 0

**Intent Performance** (tracked per intent):
- Comparison: 80% GOOD (best)
- Conditional: 75% GOOD
- Temporal: 66.7% GOOD
- Integration Chain: 50% GOOD (improvement opportunity)

---

## Conclusion

The hybrid LLM-guided query decomposition architecture demonstrates that **combining semantic understanding with deterministic validation delivers production-quality AI systems**:

**Achievements**:
- 4.4× quality improvement (15% → 66.7%)
- 2× precision improvement (50% → 100%)
- 7 operational intent types
- Production latency (3.95s, 21% under target)
- Zero regressions, 100% precision maintained

**Key Innovation**: LLMs augmenting deterministic logic beats either approach alone.

**Foundation for Future**: Threshold tuning, additional intents, latency optimization, advanced retrieval.

The pattern-based to LLM-guided evolution proves that **hybrid architectures are the path to production AI systems**.

---

**Document Type**: Portfolio Summary (Executive)  
**Detailed Version**: See private repository for comprehensive technical documentation  
**Cross-References**:
- Detailed ADR: `ADR-013-LLM-Guided-Query-Decomposition.md` (private repo)
- Journey narrative: `phase-13-14-journey-PORTFOLIO.md`
- Metrics analysis: `phase-13-14-metrics-PORTFOLIO.md`
- Related: ADR-012 (Multi-Hop Reasoning), ADR-011 (Performance Optimization)

**Status**: Accepted ✅  
**Implementation**: Complete (2026-02-20)  
**Grade**: A+ (external validation)  
**Next Review**: End of next phase (March 2026)

---

*This ADR demonstrates systematic AI engineering: incremental development, empirical validation, hybrid architectures, and production-quality deployment. For technical interviews or detailed discussions, comprehensive documentation is available in the private repository.*
