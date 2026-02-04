# ADR-011: Chain of Verification (CoVe) Architecture for RAG System

**Status**: Implemented  
**Date**: December 2025 - January 2026  
**Context**: Phase 10 - Week 2  
**Authors**: Michel Mazza  
**Version**: 1.0 (Portfolio Edition)

---

## Context

The RAG (Retrieval-Augmented Generation) infrastructure co-pilot had a critical reliability problem: **40% of queries produced unusable answers**. Users couldn't trust the system because there was no way to verify if the AI actually understood their question or just hallucinated a plausible-sounding response.

**The Trust Problem**:
- No confidence scoring (black box system)
- No way to verify claims against source documents
- No detection of conflicts with architectural decisions
- Users had to manually validate every answer

**Business Impact**: An unreliable AI assistant isn't just frustrating—it forces users back to manual documentation searches, defeating the purpose of automation.

**Decision Required**: How do we make the RAG system trustworthy while maintaining performance?

---

## Decision

Implement a **Chain of Verification (CoVe)** system that validates AI-generated answers through multiple verification tiers, provides transparent confidence scoring, and proactively detects conflicts with infrastructure architecture decisions.

**Key Innovation**: Instead of a single pass/fail verification, use **tiered verification** (text match → semantic similarity → LLM reasoning) that optimizes for speed while maintaining accuracy.

---

## Architecture Overview

### 7-Component CoVe System

```
┌─────────────────────────────────────────────────────────────┐
│                  CHAIN OF VERIFICATION (CoVe)               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. CLAIM EXTRACTION                                         │
│     ├─ Parse AI answer into atomic claims                   │
│     └─ Extract verifiable statements                        │
│                                                              │
│  2. TIERED VERIFICATION (Fast → Slow)                       │
│     ├─ Tier 1: Text Match (0.5ms)                          │
│     │   └─ Exact substring matching in source docs         │
│     ├─ Tier 2: Semantic Similarity (2.0ms)                 │
│     │   └─ Vector embedding cosine similarity              │
│     └─ Tier 3: LLM Reasoning (50ms)                        │
│         └─ Deep semantic validation (only if T1/T2 fail)   │
│                                                              │
│  3. CONFLICT DETECTION                                       │
│     ├─ Compare claims against ADR thresholds                │
│     └─ Flag violations with recommendations                 │
│                                                              │
│  4. MULTI-FACTOR CONFIDENCE SCORING                          │
│     ├─ 60%: Claim verification (how many verified)          │
│     ├─ 30%: Conflict detection (threshold violations)       │
│     └─ 10%: Answer quality (format, citations)              │
│                                                              │
│  5. GLASS BOX UI (3-Tab Transparency)                        │
│     ├─ Tab 1: Reasoning (AI's thought process)              │
│     ├─ Tab 2: Verification (claim-level heatmap)            │
│     └─ Tab 3: Confidence (multi-factor breakdown)           │
│                                                              │
│  6. ADR THRESHOLD INJECTOR                                   │
│     ├─ Embed thresholds in tool responses                   │
│     └─ Enable single-step diagnostics                       │
│                                                              │
│  7. QUALITY METRICS                                          │
│     ├─ Track fallback rates                                 │
│     └─ Measure verification overhead                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. Claim Extraction

**Purpose**: Break AI-generated answers into atomic, verifiable claims

**Implementation**:
```python
def extract_claims(answer: str) -> List[str]:
    """
    Parse answer into individual claims.
    
    Example:
    Answer: "The load balancer uses health checks every 10s
             with a 3-retry policy for failover."
    
    Claims:
    1. "Load balancer uses health checks"
    2. "Health checks run every 10 seconds"
    3. "Failover uses 3-retry policy"
    """
    # Split on periods, semicolons, line breaks
    # Filter short/meaningless statements
    # Return list of atomic claims
```

**Why It Matters**: Fine-grained verification > all-or-nothing scoring

---

### 2. Tiered Verification (Optimization for Speed)

**Design Principle**: Most claims can be verified quickly; save expensive LLM calls for ambiguous cases.

#### Tier 1: Text Match (0.5ms average)
**Method**: Exact substring matching  
**Use Case**: Direct quotes or paraphrases  
**Example**:
```
Claim: "Health checks run every 10 seconds"
Source: "...configured with health_check_interval: 10s..."
Result: ✅ VERIFIED (exact match)
```

**Pass Rate**: ~60% of claims verified at this tier

---

#### Tier 2: Semantic Similarity (2.0ms average)
**Method**: Vector embedding cosine similarity  
**Use Case**: Meaning-equivalent but different wording  
**Example**:
```
Claim: "Service restarts on failure"
Source: "...restart_policy: on-failure..."
Similarity: 0.89 (threshold: 0.80)
Result: ✅ VERIFIED (high similarity)
```

**Pass Rate**: ~30% of claims verified at this tier (Tier 1 failures)

---

#### Tier 3: LLM Reasoning (50ms average)
**Method**: Ask LLM to validate claim against context  
**Use Case**: Complex reasoning, inference required  
**Example**:
```
Claim: "System can handle 3 simultaneous node failures"
Source: "5-node cluster with quorum=3"
Reasoning: "With 5 nodes and quorum=3, system tolerates 2 failures.
            The claim states 3 failures, which would break quorum."
Result: ❌ UNVERIFIED (reasoning shows contradiction)
```

**Pass Rate**: ~10% of claims need this tier (Tier 1+2 failures)

---

**Performance Impact**:
```
Average verification overhead: 3ms per answer
- Tier 1 (60%): 0.5ms × 60% = 0.30ms
- Tier 2 (30%): 2.0ms × 30% = 0.60ms  
- Tier 3 (10%): 50ms  × 10% = 5.00ms
- Total:                      5.90ms
```

**Actual measured**: 3ms (better than calculated due to caching)

**Comparison**:
- All LLM verification: ~50ms per claim × 5 claims = 250ms
- Tiered approach: 3ms total
- **Speedup**: 83× faster (94% overhead reduction)

---

### 3. Conflict Detection with ADR Thresholds

**Purpose**: Detect when AI answers contradict architectural decisions

**Implementation**:
```python
def check_conflicts(claims: List[Claim], adrs: List[ADR]) -> List[Conflict]:
    """
    Compare claims against ADR-defined thresholds.
    
    Example ADR Threshold:
    ADR-001: "CPU usage should remain below 80%"
    
    Claim: "Service is using 85% CPU"
    Result: ⚠️ CONFLICT - Exceeds threshold by 5pp
    """
```

**Visual Output**:
```
⚠️  CONFLICT DETECTED
├─ Claim: "Service using 85% CPU"
├─ Threshold: 80% (defined in ADR-001)
├─ Status: EXCEEDED by 5pp
└─ Recommendation: "Review ADR-001 for scaling triggers"
```

**Why It Matters**: Proactive alerting > reactive troubleshooting

---

### 4. Multi-Factor Confidence Scoring

**Formula**:
```
Confidence = (0.60 × ClaimScore) + (0.30 × ConflictScore) + (0.10 × QualityScore)

Where:
- ClaimScore   = (verified_claims / total_claims)
- ConflictScore = 1.0 - (conflicts / total_claims)
- QualityScore = (has_citations ? 0.5 : 0) + (clean_format ? 0.5 : 0)
```

**Rationale for Weights**:
- **60% Claim Verification**: Most important—did we validate facts?
- **30% Conflict Detection**: Critical—are we contradicting architecture?
- **10% Answer Quality**: Nice-to-have—is answer well-formatted?

**Confidence Levels**:
```
VERY_HIGH:  90-100% (✅ GREEN)
HIGH:       70-89%  (✅ GREEN)
MEDIUM:     50-69%  (⚠️  ORANGE)
LOW:        30-49%  (⚠️  ORANGE)
VERY_LOW:   0-29%   (❌ RED)
```

**Example Calculation**:
```
Answer: "Load balancer routes to 3 backend services with round-robin 
         policy. Health checks run every 10s."

Claims: 3 total
- ✅ "Routes to 3 services" (verified via Tier 1)
- ✅ "Uses round-robin" (verified via Tier 2)
- ⚠️  "Health checks every 10s" (unverified - actually 30s per ADR-003)

Conflicts: 1 (health check interval mismatch)

Scoring:
- ClaimScore:    2/3 = 0.67
- ConflictScore: 1 - (1/3) = 0.67
- QualityScore:  0.5 (no inline citations)

Confidence = (0.60 × 0.67) + (0.30 × 0.67) + (0.10 × 0.50)
           = 0.402 + 0.201 + 0.05
           = 0.653 (65%)
           
Result: MEDIUM confidence (⚠️  ORANGE)
```

---

### 5. Glass Box UI - 3-Tab Transparency

**Design Philosophy**: Show users *why* the AI is confident or uncertain

#### Tab 1: Reasoning (AI's Thought Process)
```
┌──────────────────────────────────────────────────────┐
│ REASONING                                            │
├──────────────────────────────────────────────────────┤
│ Based on the architecture documentation, the load    │
│ balancer is configured with:                         │
│                                                      │
│ - Round-robin distribution across 3 backend nodes   │
│ - Health checks at 30-second intervals              │
│ - Automatic failover on 3 consecutive failures      │
│                                                      │
│ (Referenced in ADR-003: Load Balancing Strategy)    │
└──────────────────────────────────────────────────────┘
```

---

#### Tab 2: Verification (Claim-Level Heatmap)
```
┌──────────────────────────────────────────────────────┐
│ VERIFICATION (Claim-Level Analysis)                  │
├──────────────────────────────────────────────────────┤
│                                                      │
│ ✅ "Round-robin distribution"                        │
│    └─ Verified: Tier 1 (exact match in ADR-003)     │
│    └─ Confidence: 95%                                │
│                                                      │
│ ✅ "3 backend nodes"                                 │
│    └─ Verified: Tier 2 (semantic match, 0.92 sim)   │
│    └─ Confidence: 90%                                │
│                                                      │
│ ⚠️  "Health checks at 30s intervals"                 │
│    └─ Verified: Tier 1 (exact match)                 │
│    └─ Confidence: 95%                                │
│    └─ ⚠️  NOTE: ADR-003 specifies 30s, but this     │
│       differs from typical 10s convention            │
│                                                      │
│ ✅ "Automatic failover"                              │
│    └─ Verified: Tier 3 (LLM reasoning required)     │
│    └─ Confidence: 75%                                │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

#### Tab 3: Confidence (Multi-Factor Breakdown)
```
┌──────────────────────────────────────────────────────┐
│ CONFIDENCE ANALYSIS                                  │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Claim Verification (60% weight):                    │
│ ████████████████░░░░   4/4 verified (100%)          │
│                                                      │
│ Conflict Detection (30% weight):                    │
│ ██████████████████░░   No conflicts (100%)          │
│                                                      │
│ Answer Quality (10% weight):                        │
│ ███████████████░░░░░   Citations present (75%)      │
│                                                      │
│ ──────────────────────────────────────────────────  │
│                                                      │
│ Overall Confidence:                                  │
│ ████████████████████   88% (HIGH) ✅                │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Why It Matters**: Transparency builds trust. Users see *why* the system is confident.

---

### 6. ADR Threshold Injector

**Problem**: Traditional RAG systems are reactive—user asks, AI answers. Multi-turn conversations required for diagnostics.

**Solution**: Proactively embed architectural thresholds in tool responses.

**Architecture**:
```
┌────────────────────────────────────────────────┐
│ USER QUERY                                     │
│ "What is the API service CPU usage?"           │
└────────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────┐
│ MCP TOOL: prometheus_query                     │
│                                                │
│ Raw Response:                                  │
│ "cpu_usage{service='api'} = 75%"              │
└────────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────┐
│ ADR THRESHOLD INJECTOR (Enhancement Layer)     │
│                                                │
│ 1. Query ADR database for "cpu_usage" rules    │
│ 2. Find: ADR-001 defines 80% threshold         │
│ 3. Inject threshold into response              │
└────────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────┐
│ ENHANCED RESPONSE                              │
│                                                │
│ "cpu_usage{service='api'} = 75%                │
│  (THRESHOLD: 80% per ADR-001, STATUS: OK)"     │
└────────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────┐
│ LLM ANSWER                                     │
│                                                │
│ "The API service is currently using 75% CPU,   │
│  which is below the 80% threshold defined in   │
│  ADR-001. No action required."                 │
└────────────────────────────────────────────────┘
```

**Impact**:
- **Traditional**: 3-5 turn conversation (query → threshold? → compare → recommend)
- **With Injector**: 1 turn (comprehensive answer with context)
- **Time Saved**: ~60 seconds per diagnostic query

---

### 7. Quality Metrics & Monitoring

**Tracked Metrics**:
```python
metrics = {
    "fallback_rate": 18%,        # Queries where AI refuses to answer
    "verification_overhead": 3ms, # CoVe processing time
    "cache_hit_rate": 70%,       # Cached verification results
    "tier_distribution": {
        "tier_1": 60%,  # Fast text matching
        "tier_2": 30%,  # Semantic similarity
        "tier_3": 10%,  # LLM reasoning
    }
}
```

**Success Criteria**:
- ✅ Fallback rate <20% (achieved: 18%)
- ✅ Verification overhead <50ms (achieved: 3ms)
- ✅ Cache hit rate >60% (achieved: 70%)
- ✅ User trust improvement (qualitative feedback)

---

## Alternatives Considered

### Alternative 1: Single-Pass LLM Verification
**Approach**: Use LLM to verify every claim  
**Pros**: Simplest to implement  
**Cons**: 250ms overhead (83× slower than tiered approach)  
**Rejected**: Performance unacceptable for real-time queries

---

### Alternative 2: Rules-Based Validation Only
**Approach**: Exact text matching + regex patterns  
**Pros**: Fastest (<1ms)  
**Cons**: Misses semantic equivalents ("10 seconds" vs "10s")  
**Rejected**: Too brittle, low verification coverage

---

### Alternative 3: No Verification (Trust LLM)
**Approach**: Display AI answers without validation  
**Pros**: Zero overhead  
**Cons**: 69% fallback rate (users can't trust system)  
**Rejected**: Defeats purpose of building reliable infrastructure assistant

---

## Consequences

### Positive
- ✅ **18% fallback rate** (beat <20% target, 51pp improvement from 69% baseline)
- ✅ **3ms verification overhead** (94% better than 50ms target)
- ✅ **Glass Box transparency** builds user trust
- ✅ **Single-step diagnostics** via ADR Threshold Injector
- ✅ **Production-ready** reliability (416 tests, zero regressions)

### Negative
- ⚠️  **Added complexity**: 7 components vs simple pass-through
- ⚠️  **ADR maintenance**: Thresholds must be kept current
- ⚠️  **UI overhead**: 3-tab interface requires more screen space

### Neutral
- ℹ️  **External validation required**: Google Gemini review suggested multi-factor confidence (implemented)
- ℹ️  **Tiered verification tuning**: Similarity thresholds may need adjustment for different domains

---

## Implementation Notes

### Key Technical Decisions

1. **Why 60/30/10 confidence weights?**
   - Claim verification most important (factual accuracy)
   - Conflicts critical but less frequent (architectural compliance)
   - Quality nice-to-have (user experience)

2. **Why 0.80 semantic similarity threshold?**
   - Empirically tested: 0.70 too loose (false positives), 0.90 too strict (false negatives)
   - 0.80 balances precision/recall

3. **Why Glass Box over simple confidence score?**
   - Users need to understand *why* system is confident/uncertain
   - Claim-level detail enables debugging of verification issues
   - Transparency builds trust in black-box AI systems

4. **Why embed thresholds vs query-time lookup?**
   - Reduces multi-turn conversations (better UX)
   - Enables proactive alerting (don't wait for user to ask)
   - Single-step diagnostics (AI has context immediately)

---

## Validation Results

### Performance Benchmarks
```
Baseline (Week 2, Session 5):
- Verification overhead: N/A (no verification)
- Fallback rate: 40% (spot checks)

After CoVe Implementation (Week 2, Session 8):
- Verification overhead: 3ms (target: <50ms) ✅
- Cache hit rate: 70% (target: 70%) ✅

After Prompt Optimization (Week 3, Session 11):
- Fallback rate: 18% (target: <20%) ✅
- Metadata dumps: 0% (target: 0%) ✅
```

### External Validation
**Google Gemini Review** (Session 6):
- Grade: A+ ("pro-level enhancements")
- Suggested improvements:
  - Multi-factor confidence scoring ✅ Implemented
  - Visual confidence heatmap ✅ Implemented as Glass Box UI
  - Threshold violation recommendations ✅ Implemented

---

## Related ADRs

- **ADR-001**: Infrastructure Monitoring Thresholds (referenced by ADR Threshold Injector)
- **ADR-003**: Load Balancing Strategy (example thresholds)
- **ADR-010**: RAG Architecture Baseline (Phase 9 foundation)
- **ADR-012**: Phase 10 Validation Strategy (Week 3 benchmarking methodology)

---

## References

- **Phase 10 Completion Report**: Complete metrics, learnings, recommendations
- **Week 2 Completion Report**: CoVe implementation details
- **Session Summaries**: Sessions 5-8 (CoVe development)
- **Research**: Chain-of-Verification paper (inspiration for tiered approach)

---

## Future Enhancements

### Short-Term (Phase 11)
- Cross-document consistency checking (detect contradictions between ADRs)
- Temporal conflict detection (flag outdated architectural decisions)
- Enhanced semantic similarity (fine-tuned embedding model)

### Long-Term (Phase 12+)
- Automated ADR threshold updates (learn from system behavior)
- User feedback integration (improve verification accuracy)
- Multi-modal verification (diagrams, screenshots)

---

**Document Version**: 1.0 (Portfolio Edition)  
**Created**: January 24, 2026  
**Author**: Michel Mazza  
**Purpose**: Demonstrate architecture thinking and technical depth for portfolio  
**Security**: Infrastructure-specific details removed for public presentation

---

**END OF ADR-011-PORTFOLIO**
