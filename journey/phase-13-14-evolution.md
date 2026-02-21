# Query Decomposition Evolution: Pattern-Based to LLM-Guided Intelligence

**Portfolio Summary**: Phase 13 + Phase 14  
**Timeline**: January - February 2026  
**Achievement**: 4.4× improvement in query decomposition quality  
**Status**: Production deployment

> **Note**: This is the portfolio version (executive summary). For comprehensive technical details, see the private repository version.

---

## Executive Summary

This portfolio piece documents the evolution of an agentic RAG system's query decomposition layer from rigid pattern matching to LLM-guided intelligence, achieving a **4.4× improvement in quality** while maintaining **100% precision**.

**Key Results**:
- GOOD rate: 15% → 66.7% (4.4× improvement)
- Precision: 50% → 100% (2× improvement)
- Intent coverage: 0 → 7 operational intent types
- Pattern fallbacks: 100% → 0% (complete LLM replacement)
- Average latency: 3.95s (21% under 5s production target)

**Grade**: A+ (external validation)

---

## The Challenge

The RAG system could handle complex multi-hop queries and validate answers with Chain of Verification, but query decomposition was a critical weakness.

### The 15% Problem

Only **15% of decomposed queries** generated helpful sub-questions. The pattern-based system would turn:

**User query**: "Compare system accuracy in Version 1 vs Version 2"

**Pattern output** (POOR quality):
- "What is Version 1?"
- "What is Version 2?"
- "What is accuracy?"

These syntactically correct but semantically weak sub-questions missed the user's actual intent.

### Root Causes

**Rigid templates**: Fixed patterns couldn't adapt to query variations  
**Context blindness**: System focused on keywords, missed relationships  
**Entity obsession**: Captured nouns, ignored meaning  
**Low precision**: 50% of decompositions added no retrieval value

The validation framework measured three criteria:
1. Entity coverage (do sub-questions reference key concepts?)
2. No duplication (avoid asking the same thing twice)
3. Quality tiers (GOOD/FAIR/POOR rating)

Phase 13 baseline: 30% decomposition rate, 15% GOOD rate, 50% precision.

---

## Pattern-Based Approach (Phase 13)

Phase 13 improved entity extraction and pattern matching:

### Entity-Aware Patterns

Built a 113-term domain vocabulary and multi-word phrase detection to extract concepts like "multi-hop reasoning" instead of just "reasoning."

### Template-Based Generation

Matched query patterns to sub-question templates:

**Temporal pattern** (evolution, timeline):
```
"What was {feature} in {timeframe1}?"
"What was {feature} in {timeframe2}?"  
"How did {feature} evolve between timeframes?"
```

**Comparison pattern** (contrasting alternatives):
```
"What are characteristics of {option1}?"
"What are characteristics of {option2}?"
"How do {option1} and {option2} differ?"
```

### Three-Layer Validation

1. **Entity coverage**: Sub-questions must reference original query entities
2. **Duplication detection**: Cosine similarity (threshold: 0.85) catches duplicates
3. **Quality assessment**: GOOD (actionable) / FAIR (useful) / POOR (unhelpful)

### Results: Still 15%

Despite improvements, results were disappointing:
- Decomposition rate: 30% (up from ~10%)
- GOOD rate: 15% (unchanged)
- Precision: 50% (unchanged)

**Example failure**:
- Query: "What monitoring capabilities were added in Version 3?"
- Generated: "What is monitoring?" / "How do capabilities work?" / "What is Version 3?"
- Rating: POOR (too generic, missed context)

The pattern-based approach had hit its ceiling.

---

## The Pivot: LLM-Based Intent Classification

Phase 14 Week 1 introduced a critical shift: use LLMs to understand intent, then leverage patterns as guidance.

### Intent Taxonomy

Defined 7 intent types based on user goals:

1. **Temporal**: Evolution, timeline changes
2. **Aggregation**: Synthesis, overview
3. **Conditional**: If-then scenarios
4. **Comparison**: Contrasting alternatives
5. **Process flow**: Sequences, workflows
6. **Integration chain**: Component interactions
7. **Diagnostic**: Troubleshooting, root cause

Each intent requires different decomposition strategies.

### LLM Classification

Prompt example:
```
Classify this query's primary intent:
"{user_query}"

Intent types: temporal, aggregation, conditional, comparison, 
process_flow, integration_chain, diagnostic

Return ONLY the intent type.
```

**Validation results** (20 queries):
- Accuracy: 85% (17/20 correctly classified)
- Consistency: 100% (same query → same intent)

**Example**:
- Query: "How did system accuracy improve from V1 to V2?"
- Pattern match: temporal (keyword "improve")
- LLM classification: **comparison** (contrasting approaches) ✅
- Impact: Correct intent → better sub-questions

### Hybrid Architecture

The breakthrough wasn't LLM-only or pattern-only - it was combining both:

1. **LLM classifies intent** → semantic understanding
2. **Pattern provides template** → structural guidance
3. **LLM generates sub-questions** → nuanced output with entity context
4. **Patterns validate quality** → entity coverage, duplication checks

This architecture combined LLM flexibility with deterministic reliability.

---

## LLM-Guided Implementation

### Week 2: The MVP (4 Intents)

Implemented LLM-guided sub-question generation for 4 intent types: temporal, aggregation, conditional, comparison.

**Prompt structure**:
```
Generate sub-questions for: "{query}"

Intent: {intent_type}
Key entities: {extracted_entities}

Requirements:
1. Reference specific entities from query
2. Be actionable (answerable by documentation)
3. Build toward answering original question
4. Avoid duplication

Return sub-questions, one per line.
```

**Results** (20 queries):
- GOOD rate: 50% (vs 15% baseline) → **3.3× improvement**
- Precision: 100% (vs 50% baseline) → **2.0× improvement**
- Zero POOR decompositions

**Example success**:
- Query: "How do components A, B, and C work together?"
- Intent: integration_chain
- Generated:
  - "What role does component A play in the system?"
  - "How does component B integrate with component A?"
  - "How does component C complete the integration chain?"
- Rating: GOOD (captures component interactions)

Week 2 delivered **74% of total improvement** (35pp out of 47.3pp).

### Week 3: Scaling to 7 Intents

Expanded to full 7-intent system with comprehensive testing.

**Added intents**:
- Process flow: Deployment workflows, sequences
- Integration chain: Component interactions
- Diagnostic: Troubleshooting, root cause

**Comprehensive validation** (30 queries):
- 10 from baseline (comparison)
- 10 new queries (coverage validation)
- 10 edge cases (stress testing)

**Results**:
- GOOD rate: 66.7% (vs 15% baseline) → **4.4× improvement**
- Precision: 100% (maintained)
- Intent coverage: 7/7 operational
- No quality loss from expansion

---

## Results: 4.4× Quality Improvement

### Primary Metrics

| Metric | Phase 13 | Phase 14 | Improvement |
|--------|----------|----------|-------------|
| **GOOD Rate** | 15% | 66.7% | **4.4×** |
| **Precision** | 50% | 100% | **2.0×** |
| **Intent Types** | 0 | 7 | **+7** |
| **POOR Rate** | 31% | 0% | **-31pp** |
| **Avg Latency** | N/A | 3.95s | **21% under target** |

### Quality Distribution Evolution

**Phase 13**:
- GOOD: 15%
- FAIR: 54%
- POOR: 31%

**Phase 14**:
- GOOD: 66.7%
- FAIR: 33.3%
- POOR: 0%

**Achievement**: Complete elimination of POOR decompositions.

### Real-World Example

**Query**: "How did system accuracy improve from V1 to V2, and what techniques were added?"

**Phase 13 (Pattern-based - POOR)**:
- "What is V1?"
- "What is V2?"
- "What techniques exist?"

**Phase 14 (LLM-guided - GOOD)**:
- "What was system accuracy in V1?"
- "What accuracy did V2 achieve?"
- "What techniques were introduced between V1 and V2?"

The LLM captured comparison intent and generated entity-rich, actionable sub-questions.

---

## Key Learnings

### 1. Hybrid Architectures Win

Don't replace deterministic logic with LLMs - augment it.

**LLMs excel at**: Semantic understanding, nuanced generation, edge cases  
**Patterns excel at**: Structure, performance, explainability  
**Hybrid approach**: 4.4× improvement + 100% precision

### 2. MVP Approach Validated

Built incrementally: 4 intents → 7 intents.

**Why it worked**:
- Validated early (50% GOOD rate at 4 intents)
- Failed fast (if 4 failed, know before investing in 7)
- Built confidence (no quality loss from expansion)

### 3. Conservative Thresholds Prevent Degradation

Used 0.5 confidence threshold (50% minimum).

**Trade-off**:
- Benefit: Zero POOR decompositions (100% precision)
- Cost: ~20% of queries blocked

**Phase 15 opportunity**: Lower to 0.3-0.4 could unlock +10-15pp GOOD rate.

### 4. Validation Layers Essential

Three-layer validation prevented quality issues:
- Entity coverage (prevents generic questions)
- Duplication detection (efficiency)
- Quality assessment (GOOD/FAIR/POOR rating)

Result: 50% → 100% precision.

### 5. Entity Context Makes LLMs Smarter

**Without entities**: "What is the feature?" (generic)  
**With entities**: "How does {specific feature} work in {context}?" (actionable)

Entity extraction isn't just for patterns - it's essential for LLM quality.

### 6. Measure Twice, Cut Once

Every decision was data-driven:

**Week 1**: Test LLM classification → 85% accuracy → proceed  
**Week 2**: Test 4 intents → 50% GOOD (3.3× improvement) → proceed  
**Week 3**: Test 7 intents → 66.7% GOOD (4.4× improvement) → ship

No guessing, only measuring.

### 7. The 80/20 Rule in AI

Week 2 delivered 74% of total improvement (35pp out of 47.3pp).

**Lesson**: The first implementation of a good idea delivers most value. Refinements add incremental gains.

### 8. Production Quality Requires Discipline

Maintained zero regressions and 100% precision across three weeks.

**How**: Comprehensive testing, validation layers, conservative thresholds.

**Lesson**: Production quality isn't an accident - it's systematic engineering.

---

## What's Next

The Phase 14 foundation enables several enhancements:

### Confidence Threshold Tuning (High Priority)

**Current**: 0.5 threshold blocks ~20% of queries  
**Opportunity**: Lower to 0.3-0.4 could unlock +10-15pp GOOD rate  
**Risk**: Low (validation layers prevent degradation)

### Additional Intent Types

**Candidates**: Ranking, summarization, exploration  
**Foundation**: 7-intent expansion showed no quality loss  
**Approach**: Incremental (1-2 intents per phase)

### Advanced Retrieval Optimization

**Focus**: Algorithm tuning, evaluation frameworks, hybrid search  
**Foundation**: High-quality decomposition unlocks retrieval improvements

The 4.4× improvement in query decomposition quality unlocks the next level of RAG system performance.

---

## Architecture Highlights

### System Components

1. **Intent Classifier**: LLM-based semantic understanding
2. **Entity Extractor**: 113-term domain vocabulary + phrase detection
3. **Sub-Question Generator**: LLM with pattern guidance
4. **Validation Pipeline**: 3-layer quality assurance
5. **Quality Assessor**: LLM-based GOOD/FAIR/POOR rating

### Technology Stack

- **LLM**: Local inference (8B parameter model)
- **Embeddings**: 384-dimensional semantic vectors
- **Validation**: Cosine similarity, entity coverage heuristics
- **Latency**: 3.95s average (sub-second classification, 2.5s generation)

### Production Deployment

- **Confidence threshold**: 0.5 (conservative)
- **Precision**: 100% (zero POOR decompositions)
- **Regression testing**: 454/465 tests passing (97.6%)
- **External validation**: A+ grade

---

## Conclusion

The evolution from pattern-based to LLM-guided query decomposition demonstrates that **hybrid architectures deliver production-quality AI systems**:

**Achievements**:
- 4.4× quality improvement (15% → 66.7% GOOD rate)
- 2.0× precision improvement (50% → 100%)
- 7 operational intent types
- Zero pattern fallbacks
- Production latency (3.95s, 21% under target)

**Key Innovation**: Combining LLM semantic understanding with deterministic validation creates systems that are both intelligent and reliable.

The pattern-based to LLM-guided journey proves that the future of AI systems isn't LLMs replacing logic - it's LLMs augmenting it.

---

**Document Type**: Portfolio Summary (Executive)  
**Detailed Version**: See private repository for comprehensive technical documentation  
**Cross-References**:
- Detailed narrative: `phase-13-14-journey.md` (private repo)
- Comprehensive metrics: `phase-13-14-metrics.md` (private repo)
- Phase 13 completion report (private repo)
- Phase 14 completion report (private repo)

**Timeline**: January - February 2026  
**Achievement**: 4.4× improvement in query decomposition quality  
**Grade**: A+ (external validation)  
**Status**: Production deployment complete

---

*This portfolio piece demonstrates systematic AI engineering methodology: incremental development, empirical validation, hybrid architectures, and production-quality deployment. For technical interviews or detailed discussions, comprehensive documentation is available in the private repository.*
