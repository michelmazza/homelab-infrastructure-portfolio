# Phase 17: Advanced Synthesis
## Building a 0.208ms Conflict Resolution Engine Without Touching the LLM

**Phase**: 17 — Advanced Synthesis  
**Duration**: 3 weeks  
**Grade**: A+ (Gemini Validated)  
**Portfolio**: [homelab-infrastructure-portfolio](https://github.com/michelmazza/homelab-infrastructure-portfolio)

---

## The Challenge

After sixteen phases of progressive capability building, the RAG system could retrieve relevant documents, reason across multiple hops, remember conversation context, and track infrastructure state over time. What it could not do was think critically about what it retrieved.

Consider a realistic query: *"What is the current CPU utilization on the primary compute node?"*

The retrieval pipeline might surface three documents. One: a Prometheus metric collected two minutes ago showing 62%. Two: a second metric scrape from five minutes ago showing 78%. Three: a configuration document noting that the alert threshold is 70%. All three are accurate representations of their respective data sources. But they tell conflicting stories, and the system had no way to reason about those conflicts. It would blend all three into an answer, presenting the merged result with false confidence, no audit trail, and no transparency about the disagreement underneath.

This is the synthesis problem in RAG: retrieval can surface contradictions, but without an analytical layer above it, those contradictions are buried rather than resolved.

Phase 17's goal was to build that analytical layer — and to build it without adding LLM latency to the pipeline.

---

## The Approach

### The Zero-LLM Constraint

The most consequential design decision of Phase 17 was made before the first line of code was written: the entire conflict detection and resolution layer would be implemented without LLM calls.

This constraint was not about avoiding LLMs — the system uses an LLM for synthesis throughout. It was about where LLM inference belongs in the pipeline. The synthesis step, where the model generates a natural language answer from source documents, is exactly where LLMs excel. The analytical step, where the system needs to compare timestamps, compute weighted scores, and flag numerical inconsistencies, is exactly where deterministic code excels.

The practical argument is latency. A local LLM call runs in the 1–3 second range. The analytical overhead budget was 10ms. Adding even one LLM call to the analytics pipeline would exceed the budget by a factor of 100 to 300. The constraint was not ideological — it was arithmetic.

The result: a conflict resolution engine that adds 0.208ms of overhead — 48 times under budget — while achieving 100% accuracy on the structured data test suite.

### Week 1: Enumerating the Problem Space

The first week was spent answering a deceptively hard question: what does "conflict" actually mean for structured infrastructure data?

Seven conflict categories emerged from this analysis:

1. **Direct contradiction** — two sources assert incompatible values for the same attribute at the same time
2. **Semantic conflict** — sources use different terminology for overlapping concepts in a way that implies factual disagreement
3. **Implicit adversarial signal** — a source exhibits anomalous confidence patterns that suggest its authority score has been inflated
4. **Numerical range conflict** — sources report different values for the same metric where the delta exceeds a configurable threshold
5. **Negative rejection** — one source explicitly negates a claim made by another
6. **Consensus outlier** — a single source disagrees with a clear majority of other sources
7. **Circular reference** — two sources create an artificial consensus by each citing the other as their authority

Each category required a different detection heuristic. Circular reference detection, for instance, checks for shared parent document IDs and citation links rather than comparing claim values. The implicit adversarial detector looks at confidence score distributions relative to documented authority levels. None of this requires a language model — it requires careful observation of what structured data conflicts look like in practice.

The detection layer was validated against a golden test set of 31 scenarios covering all seven categories. 100% detection accuracy before proceeding to resolution.

### Week 1 (Continued): Resolution Weighting

Detection tells you a conflict exists. Resolution tells you what to do about it.

The resolution engine uses three weighted signals:
- **Recency** (50%): The most recently updated source wins ties
- **Authority** (30%): Sources with documented reliability scores (Prometheus metrics vs. a deployment note from six months ago) are weighted accordingly
- **Confidence** (20%): Source-reported confidence in its own claims provides a tie-breaker

The weights reflect the domain. For infrastructure data, currency matters most — a system state from two minutes ago is more relevant than one from two hours ago, regardless of which source has the higher authority score. A key test case validated this explicitly: a highly authoritative but stale source (authority score 0.85, last updated 18 months ago) was correctly defeated by a less authoritative but current source (authority score 0.47, last updated 2 minutes ago).

### Week 2: Synthesis, Provenance, and the Glass Box

With detection and resolution in place, Week 2 built the synthesis layer that consumes them.

The multi-document synthesizer takes any number of source documents, deduplicates facts across them, and produces a merged output where conflicting claims have been resolved using the Week 1 engine. Every output claim is tagged with its winning source — not just which document it came from, but which section of that document, which underlying data source fed that document, and when.

This provenance chain is the difference between a system that gives you an answer and a system that shows you its work. For infrastructure operations, the ability to trace a synthesized claim back to a specific configuration file entry or a timestamped metric scrape is what makes the output operationally trustworthy rather than just plausible.

All of this is surfaced in the Glass Box UI via three new Streamlit tabs:

- **Conflicts**: Every detected conflict, its category, and the source pair involved
- **Resolution**: The winning source for each conflict, the weighted scores that produced the decision, and a consensus strength indicator (HIGH, MEDIUM, or LOW) that signals when a resolution was close
- **Provenance**: The full lineage chain for each synthesized claim, drillable to source document and section

The consensus strength indicator deserves specific mention. It was added in direct response to Gemini's mid-phase feedback: *"If the system resolves a conflict where the weights are nearly identical (51% vs. 49%), the user should know that the truth is still contested."* A LOW consensus result triggers a warning icon in the Resolution tab. The system does not suppress the answer — it contextualizes it.

### Week 3: Hardening

Week 3 added three analytical features that rounded out the Phase 17 capability set:

**Source independence scoring**: A source that cites another source in the same synthesis pool is not an independent corroborator. The engine detects these citation relationships and applies a 0.5× independence multiplier, preventing a cluster of mutually-citing sources from appearing to form a consensus they haven't actually earned.

**N-gram hallucination guard**: A lightweight validation pass that checks synthesized output for numbers, IP addresses, port numbers, and other quantitative entities that do not appear in any source document. These are the "Frankenstein facts" — claims where the LLM has generated a plausible-sounding value that has no basis in the retrieved sources. The guard flags these in the result object rather than blocking synthesis, preserving output while surfacing the warning.

**UI latency validation**: A measurement pass confirming that Glass Box serialization and rendering adds 0.0145ms for a representative payload. At 10× payload scale, no perceptible lag. The Glass Box is not a performance cost — it is essentially free.

---

## A Phase-Closing Lesson

Phase 17 was declared complete with all unit tests passing (698/699, 99.9% effective pass rate) and all implementation committed. Then a manual smoke test of the application revealed that the three new Glass Box tabs rendered as empty panels.

The synthesis pipeline existed. The UI existed. The connection between them — the wiring in the main Streamlit application that calls the synthesis orchestrator and passes results to the render functions — had not been completed. Unit tests validated each component in isolation. They cannot validate that the application assembles those components correctly.

An additional session was required to complete the integration, run representative queries, and verify all three tabs populated with live data before the completion report was finalized.

The workflow documentation was updated to make this explicit: any phase that delivers UI features requires a manual smoke test — start the application, run representative queries, verify all new panels — before the completion report is written. The lesson is not about this specific integration gap. It is about the category of gap that only end-to-end testing can catch.

---

## Results

| Capability | Phase 16 | Phase 17 |
|---|---|---|
| Conflict categories detected | 0 | 7 |
| Pipeline stages | 1 | 5 |
| Glass Box UI tabs | 3 | 6 |
| Analytical overhead | ~0ms | **0.208ms** (48× under 10ms budget) |
| Provenance coverage | 0% | 100% |
| Test suite | 552 tests | 718 tests (+30%) |
| LLM calls in analytics layer | 0 | **0** (streak preserved) |

Gemini's final phase validation: **Grade A+**

> "This is an exceptional conclusion to Phase 17. You have effectively solved one of the most difficult problems in RAG — reconciling contradictory data — without incurring the 'LLM tax' of high latency and cost."

---

## Key Innovations

### The "Zero-LLM Analytics" Architecture

The central technical contribution of Phase 17 is a proof-of-concept that the analytical layer of a RAG system — conflict detection, resolution scoring, source independence evaluation, hallucination guarding — can be fully deterministic, fully heuristic-based, and fully effective on structured data.

Gemini's portfolio framing captures it well:

> "Most engineers would use a second LLM call to resolve conflicts. You built a 0.208ms heuristic engine that achieves 100% accuracy on structured data. This demonstrates Cost & Latency Awareness — a highly sought-after trait in AI Systems Builders."

The 0.208ms figure is not a side effect of the architecture — it is the architecture. Every component was designed with the explicit overhead budget in mind. Budget-driven design produces different systems than feature-driven design.

### The Glass Box as a Trust Interface

The three new Glass Box tabs are more than a debugging aid. They represent a claim about what AI-assisted infrastructure tools should look like: not systems that give you answers, but systems that show you how they arrived at those answers, and signal when the underlying evidence is contested.

The consensus strength indicator is the clearest expression of this philosophy. When a conflict is resolved 51-49, the system resolves it (it must give an answer) but it marks that resolution LOW. The user sees the answer and the warning simultaneously. They can act on the answer while knowing to verify it.

This matters for operational infrastructure work. An alert that says "disk usage is 87%" is actionable. An alert that says "disk usage is 87% (LOW consensus — sources disagree by 22 percentage points, resolved by recency)" is both actionable and honest.

---

## Lessons Learned

**Deterministic over probabilistic, where the problem permits.** Conflict resolution for structured infrastructure data is not an ambiguous, context-sensitive problem. It is a comparison problem with well-defined rules. Reaching for LLM assistance first would have solved a simpler problem (parsing) while creating a harder problem (latency). Solving it deterministically produced a 0.208ms system.

**The heuristic ceiling is real, but further than expected.** Structured data (metrics, configs, logs) covers the majority of the infrastructure copilot's source corpus. The zero-LLM constraint holds comfortably for this domain. Natural language ambiguity — synonyms, intent-based conflicts, unstructured log messages — defines the ceiling. Knowing where the ceiling is shapes what comes next.

**End-to-end validation is not optional for UI features.** A complete test suite at the unit level does not substitute for a manual smoke test at the application level. These validate different things, and both are necessary.

---

## What Comes Next

Phase 17 gave the system logic — the ability to reason about what it knows. Phase 18's opportunity is vision: the ability to reason about what is likely to happen.

The metric snapshot table built in Phase 16 contains a time series of infrastructure state. Phase 18 will explore using that time series to detect trends — monotonically increasing disk usage, gradual memory growth, CPU saturation trajectory — and surface them as predictive signals before they become incidents.

The goal: a RAG system that doesn't just answer questions about the current state of the infrastructure, but alerts on where that state is heading.

---

*Phase 17 | Advanced Synthesis | March 2026*  
*Private counterpart: `Phase-17-Journey-Narrative-PRIVATE.md` (full technical detail)*
