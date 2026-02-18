# Phase 12: From Assumptions to Evidence - A Data-Driven Retrieval Journey

**Duration**: 3 sessions (1 week, completed early)  
**Status**: ✅ Complete  
**Grade**: A (External AI validation)  
**Achievement**: 92% hit rate (+20pp) via strategic pivot

---

## Table of Contents

1. [The Challenge](#the-challenge)
2. [The Strategic Pivot](#the-strategic-pivot)
3. [Week 1: RRF Validation](#week-1-rrf-validation)
4. [Week 2: Corpus Expansion](#week-2-corpus-expansion)
5. [The Test Discovery](#the-test-discovery)
6. [Results](#results)
7. [Key Engineering Principles](#key-engineering-principles)
8. [What's Next](#whats-next)
9. [Conclusion](#conclusion)

---

## The Challenge

Phase 11 concluded with impressive multi-hop reasoning capabilities, but retrieval quality emerged as the primary bottleneck. The math was simple: with an 85% per-hop hit rate, compound multi-hop success dropped to 72% (85% × 85%). To reach our 88.5% multi-hop target, we needed 92% per-hop accuracy.

External AI validation predicted an aggressive but achievable 92% target through a three-week approach: Reciprocal Rank Fusion (Week 1), Docling structure-aware chunking (Week 2), and RAGAS systematic evaluation (Week 3).

The assumptions seemed solid. RRF would improve retrieval ranking by eliminating score normalization issues. Docling would add marginal gains through better document structure awareness. RAGAS would validate the improvements with systematic metrics.

**Then came Session 1.**

---

## The Strategic Pivot

### The 30-Minute Investigation

The baseline benchmark revealed an unexpected result: 72% hit rate instead of the expected 85%. This 13-percentage-point gap demanded investigation before implementation.

Rather than immediately building RRF to "fix" the problem, I paused. Thirty minutes of systematic investigation would prove critical.

I deep-dived into five diverse failure examples: high-score failures, threshold queries, and RAG-internal questions. The goal was simple: understand *why* retrieval was failing before deciding *how* to fix it.

### The Discovery

The investigation revealed something surprising: **43% of failures weren't retrieval problems at all—they were data gaps.**

The embedding corpus only contained Phases 4-7 documentation (611 chunks, 85 documents). Queries about Phase 8-11 features were asking about content that simply didn't exist in the database. Questions like "What is the BM25 alpha parameter?" (a Phase 8 implementation detail) returned no results because the answer literally wasn't there.

**Failure categorization broke down like this:**
- **Corpus gap**: 6/14 failures (43%) - missing content, not retrieval quality
- **Retrieval ranking**: 4/14 failures (29%) - right document exists, wrong position
- **LLM extraction**: 3/14 failures (21%) - right context retrieved, LLM fails to extract
- **Undocumented**: 1/14 failures (7%) - feature not documented anywhere

### The Adjusted Reality

When I excluded the corpus gap queries (impossible to answer), the adjusted baseline jumped to **81.8%**—almost exactly the 85% Phase 11 had estimated. Phase 11 wasn't wrong about retrieval quality; we just had incomplete data coverage.

### The Impact Analysis

Now came the critical decision: what to prioritize?

**RRF impact estimate**:
- Fixes: 4/14 failures (retrieval ranking issues)
- Expected improvement: +8 percentage points (72% → 80%)

**Corpus update impact estimate**:
- Fixes: 6/14 failures (data gaps)
- Expected improvement: +12 percentage points (80% → 92%)

**Docling impact estimate** (original Week 2 plan):
- Expected improvement: +2-3 percentage points (marginal gains)

The data was clear: corpus expansion offered **4-6× higher impact** than Docling.

### The Pivot

**Original plan:**
- Week 1: RRF (85% → 92%)
- Week 2: Docling (+2-3pp)

**Data-driven pivot:**
- Week 1: RRF (72% → 80%) ✅ Proceed
- Week 2: **Corpus expansion** (80% → 92%) ⭐ Swapped
- Week 3: RAGAS validation

**Rationale**: Fix the root cause (missing data) before optimizing the algorithm (structure-aware chunking). Docling deferred to Phase 13.

This thirty-minute investigation prevented spending an entire week on marginal improvements. **"Measure twice, cut once"** proven in practice.

---

## Week 1: RRF Validation

### Theory Meets Reality

Reciprocal Rank Fusion (RRF) should theoretically outperform α-weighted fusion by eliminating score normalization issues. The algorithm is elegant: instead of combining raw similarity scores, combine rankings. Each retrieval method votes by position, and higher positions carry more weight.

I implemented RRF with eleven comprehensive unit tests. Everything passed. Ready to deploy.

Then came the empirical validation.

### The Surprising Result

When tested on our actual corpus, **α-weighted fusion (90% accuracy) beat RRF (75-80%)** across the query set.

**Why?** Our small corpus (611 chunks at the time) worked well with score normalization. The α=0.3 parameter (30% keyword, 70% semantic) had been empirically tuned in Phase 8 to reflect our specific document characteristics. RRF's default k=60 parameter wasn't optimized for our use case.

More importantly: the score separation in α-weighted fusion provided clearer confidence signals. Top-3 results showed distinct score gaps (0.85, 0.72, 0.65) while RRF compressed differences (0.033, 0.031, 0.029).

### The Decision

I kept α-weighted fusion.

This wasn't abandoning RRF due to implementation issues—all tests passed. This was **choosing empirical evidence over theoretical frameworks**. The data showed α-weighted performed better on our corpus, so we stayed with what worked.

RRF remains implemented and tested, ready if corpus characteristics change in the future. But for Phase 12, evidence trumped theory.

**Week 1 lesson**: Don't implement "better" approaches without validation. Theoretical improvements may not translate to practical gains.

---

## Week 2: Corpus Expansion

### The Execution

With retrieval ranking validated (α-weighted kept), Week 2 focused on the highest-impact fix: expanding corpus coverage from Phases 4-7 to Phases 4-12.

**Scope**:
- Add ~185 documents from Phases 8-11
- Expected: 611 → ~1,500 chunks
- Process: Re-embed with existing pipeline

I delegated execution to Claude Code for efficiency: identify documents, extract text, chunk at 1,000 tokens with 200-token overlap, generate embeddings, merge with existing corpus.

**Execution time: 18 minutes.**

### The Results

**Corpus growth:**
- Documents: 85 → **270** (3.2× expansion)
- Chunks: 611 → **2,095** (3.4× expansion)
- Phase coverage: 4-7 → **4-12**

**Performance improvement:**
- Baseline: 72%
- Post-corpus: **92%** (+20pp)

**Corpus gap queries fixed: 5/6 (83% success rate)**

Query examples:
- ✅ "What is the BM25 alpha parameter?" - Now finds Phase 8 implementation details
- ✅ "How does intent classification work?" - Now retrieves Phase 9 agentic RAG architecture
- ✅ "What is CoVe?" - Now surfaces Phase 10 Chain of Verification documentation
- ✅ "How does multi-hop routing work?" - Now returns Phase 11 multi-hop architecture

One query (Q32: "How is BM25 combined with vector search?") remained unsolved—not a corpus issue but a retrieval scoring problem, deferred to Phase 13.

### Ahead of Schedule

The 92% target was achieved in **Week 2 instead of Week 3**. By addressing the root cause (missing data) rather than symptoms (algorithm optimization), we reached the primary objective early.

This left Week 3 for comprehensive RAGAS evaluation, which would reveal the next bottleneck.

---

## The Test Discovery

### The Blind Spot

During Session 3 regression testing, I ran the standard command: `pytest tests/ -v`

Result: **17/17 tests passed** ✅

The planning documents referenced "416 tests" as the baseline. Where were the other 399 tests?

I ran full test discovery: `pytest --collect-only -q`

**Result: 469 tests found project-wide.**

### The Structure

The 469 tests weren't in one location:
- `tests/` directory: 17 tests (what I'd found initially)
- `rag/tests/`: 24 tests (async connection/execution)
- Root-level integration tests: 10 tests (Prometheus)
- Script-level tests: 3 tests
- Parameterized expansions: 415 tests (generated from test decorators)

**Directory filtering missed 96% of the test suite.**

### The Full Validation

Running the complete suite revealed:
- **Total**: 469 tests
- **Passed**: 432 (92.1%)
- **Failed**: 37 (all pre-existing infrastructure/config issues)
- **New regressions**: 0 ✅

The 37 failures broke down into three categories:
- 24 async tests (missing `pytest-asyncio` package, never passed since Phase 10)
- 10 Prometheus integration tests (require live infrastructure)
- 3 script fixture errors (pre-existing configuration issues)

**Zero new regressions.** Week 1's RRF work and Week 2's corpus expansion introduced no failures.

### The Process Fix

I created `Testing-Guide.md`, a comprehensive testing reference documenting:
- Complete test structure (469 tests across 37 files)
- Regression testing workflow (discovery → execution → analysis)
- Known pre-existing failures (categorized and documented)
- Quick reference commands

Updated `CHAT-SESSION-WORKFLOW.md` with the regression testing standard: Session 3 must **always** run full test discovery first (`pytest --collect-only`), never filter to a single directory.

**Process improvement prevents future blind spots.** Phase 13 and beyond will have comprehensive regression validation.

---

## Results

### Primary Achievement

**Hit rate: 72% → 92% (+20pp)** via strategic corpus expansion

This matched the external AI validator's aggressive prediction and achieved the phase objective ahead of schedule (Week 2 instead of Week 3).

### Corpus Transformation

| Metric | Before | After | Growth |
|--------|--------|-------|--------|
| Documents | 85 | 270 | 3.2× |
| Chunks | 611 | 2,095 | 3.4× |
| Phase coverage | 4-7 | 4-12 | +5 phases |

### Test Coverage

| Metric | Value |
|--------|-------|
| Total tests | 469 |
| Passing | 432 (92.1%) |
| Effective rate | 100% (432/432 runnable) |
| New regressions | 0 ✅ |

### RAGAS Evaluation Insights

Week 3 implemented the RAGAS evaluation framework, revealing important methodology learnings:

**Reliable metrics** (bias-free):
- Context Entity Recall: 0.673 (query entities found in retrieved context)

**Methodology-limited metrics** (self-grading bias):
- Context Precision: 1.000 (suspiciously perfect, inflated by local LLM evaluator)
- Faithfulness: 0.567 (24/50 evaluations timed out)

**Key finding**: Same 8B model used for both generation and evaluation created self-grading bias. Phase 13 will use external API evaluator (Claude 3.5 Sonnet or GPT-4o) for reliable baselines.

### The Hidden Bottleneck

Multi-hop success rate remained at 80%—unchanged despite 92% retrieval improvement.

**Root cause**: Only 2/10 complex queries triggered multi-hop routing. The system classified 8/10 multi-aspect queries as single-hop, preventing the multi-hop engine from engaging.

**External AI validator's quote**: *"I optimized the engine (Retrieval) until I hit the limits of the transmission (Routing)."*

The bottleneck shifted from retrieval quality (fixed at 92%) to routing classification (now the constraint). Phase 13 will focus on routing intelligence rather than retrieval tuning.

### Grade: A (External Validation)

**Rationale**:
- Primary objective achieved (92% hit rate) ✅
- Corpus gap discovery demonstrated systematic debugging ("glass box win")
- α-weighted decision showed technical maturity (evidence over theory)
- Bottleneck shift identified (routing exposed as next constraint)

---

## Key Engineering Principles

### 1. Measure Twice, Cut Once

**Session 1's thirty-minute investigation prevented Week 2 being spent on marginal improvements.**

Instead of implementing RRF immediately to "fix" the 72% baseline, I investigated why the baseline differed from expectations. That investigation revealed 43% of failures were data gaps, not retrieval quality issues.

The pivot from Docling (+2-3pp) to corpus expansion (+12pp) delivered 4-6× higher impact. Strategic resource allocation based on data, not assumptions.

**Practical application**: When baselines differ significantly from expectations (>10pp), pause and investigate root causes before implementing solutions. Thirty minutes investigating can save three hours implementing the wrong fix.

### 2. Evidence Over Theory

**RRF should have been superior. The data said otherwise.**

Reciprocal Rank Fusion eliminates score normalization issues—theoretically better than α-weighted fusion. Implementation was clean, all tests passed, ready to deploy.

Empirical validation showed α-weighted (90% accuracy) outperformed RRF (75-80%) on our specific corpus. Small corpus size and pre-tuned α=0.3 parameter created conditions where score normalization worked well.

I kept α-weighted fusion despite RRF's theoretical advantages. **Evidence trumped theory.**

**Practical application**: Always validate improvements empirically before adopting. "Better in theory" doesn't guarantee "better in practice." Corpus characteristics, parameter tuning, and real-world conditions matter more than theoretical frameworks.

### 3. Comprehensive Testing Requires Full Discovery

**Planning documents referenced 416 tests. Session 3 found 17 tests. Full discovery revealed 469 tests.**

Directory filtering (`pytest tests/`) only discovered 3.6% of the test suite. The remaining 96% lived in `rag/tests/`, root-level integration tests, script tests, and parameterized expansions.

Without full test discovery, I would have validated against an incomplete baseline, missing potential regressions in 452 tests.

**Practical application**: Never assume test coverage without verification. Always run full test discovery (`pytest --collect-only`) before regression validation. Document test structure to prevent future blind spots. Created `Testing-Guide.md` to codify this lesson.

### 4. Bottlenecks Shift When You Fix One Component

**Optimized retrieval to 92%. Multi-hop remained at 80%. Why?**

Phase 11's bottleneck was retrieval quality (85% per-hop limiting multi-hop compound success). Phase 12 fixed retrieval (92%).

Expected result: Multi-hop improves to 88.5% (92% × 92%).  
Actual result: Multi-hop stayed at 80%.

Investigation revealed routing classification as the new bottleneck: only 2/10 complex queries triggered multi-hop execution. The system classified 80% of multi-aspect queries as single-hop, preventing the improved retrieval from being utilized.

**Practical application**: System optimization is iterative, not one-shot. Fix the highest-impact bottleneck, measure results, identify the next constraint. External validator's metaphor captured it perfectly: "I optimized the engine (Retrieval) until I hit the limits of the transmission (Routing)."

---

## What's Next

### Phase 13 Priorities (External AI Ranked)

**1. Multi-Hop Routing Intelligence** (Primary Focus)
- Replace word-count heuristics with semantic complexity scoring
- Improve query classification (2/10 → 8/10 complex queries correctly identified)
- Expected impact: 80% → 88.5% multi-hop success

**2. External RAGAS Evaluator** (High Priority)
- Use Claude 3.5 Sonnet or GPT-4o for evaluation
- Eliminate self-grading bias (Context Precision 1.000 → realistic scores)
- Establish reliable quality baselines for ongoing measurement

**3. Entity Extraction Completeness** (Medium Priority)
- Current: 0.673 entity recall (1-2 entities per query)
- Target: 0.85 entity recall (3-5 entities per query)
- Improves retrieval precision by capturing more query aspects

**4. Docling Structure-Aware Chunking** (Medium Priority)
- Deferred from Phase 12 Week 2 (data-driven pivot)
- Expected: +2-3pp improvement
- Focus: Table extraction, code blocks, hierarchical structure preservation

### Foundation Ready

**Strengths going into Phase 13:**
- ✅ 92% retrieval baseline (strong foundation)
- ✅ 2,095 chunk corpus (Phases 4-12 coverage)
- ✅ RAGAS framework operational (infrastructure in place)
- ✅ Clear bottleneck identified (routing classification)
- ✅ Zero regressions (432/432 tests passing)
- ✅ Comprehensive testing standard (prevents blind spots)

**Immediate Quick Wins:**
- Install `pytest-asyncio` (24 tests, 5 minutes)
- Mock Prometheus APIs (10 tests, 1-2 hours)
- Fix script fixtures (3 tests, 30 minutes)
- Target: 469/469 tests @ 100% pass rate

---

## Conclusion

Phase 12 achieved its primary objective—92% per-hop hit rate (+20pp improvement)—through systematic problem-solving and data-driven decision making.

The phase demonstrated four critical engineering practices:

1. **Investigate before implementing** - Thirty minutes of root cause analysis prevented Week 2 being spent on marginal improvements
2. **Evidence over theory** - Empirical validation showed α-weighted outperformed RRF despite theoretical advantages
3. **Comprehensive testing** - Full test discovery (469 tests) prevented incomplete regression validation
4. **Iterative optimization** - Fixing retrieval exposed routing as the next bottleneck

The corpus expanded 3.4× (611 → 2,095 chunks), covering Phases 4-12 documentation. Zero new regressions across 432 passing tests confirmed foundation stability. RAGAS evaluation framework is operational, with methodology learnings guiding Phase 13 external evaluator implementation.

**Key takeaway**: *"By scaling the corpus 3.4× and achieving 92% retrieval accuracy, I successfully removed the 'Data Noise' bottleneck. This exposed a previously hidden limitation in the system's routing classification logic."*

The engine is optimized. Now to fix the transmission.

**Phase 13 ready**: Clear priorities (routing intelligence, external evaluator), proven methodology ("measure twice, cut once"), strong foundation (2,095 chunks, zero regressions).

---

**Document Version**: 1.0 (Portfolio)  
**Created**: February 17, 2026  
**Purpose**: Interview-ready narrative for recruiters and technical interviews  
**Status**: Public portfolio content

**Counterpart**: `Phase-12-Journey-Full.md` (Private, ~1,200 lines comprehensive technical detail)

---

*This is the concise portfolio version. See private documentation for complete technical depth.*
