**Question: "How do you use AI in your development workflow?"**

Explain comprehensive AI integration:
1. Planning: Validate architecture in Session 0 with AI
2. Implementation: Get patterns and examples during coding
3. Debugging: Use AI to interpret errors and suggest fixes
4. Validation: Cross-validate with multiple AIs (Claude + Gemini)
5. Learning: Accelerate understanding of unfamiliar tech

**Example**: Phase 8 Session 0 - validated three-pillar strategy with Claude before writing code, got BM25 integration patterns, designed test suite structure. Session 3 - Gemini caught bugs Claude missed. Not just "ChatGPT for code" but systematic AI collaboration throughout lifecycle.# Engineering Methodology

How I approach complex engineering problems through systematic validation, data-driven decisions, and continuous improvement.

---

## Philosophy

**Measure twice, cut once.** Build systems that can be understood, validated, and improved through data rather than intuition. Document decisions while context is fresh. Iterate rapidly but deliberately.

**Core Principles**:
- Validation before optimization
- Data over assumptions
- Systematic over heroic
- Document while fresh
- Build, measure, learn, iterate

---

## 1. Session-Based Development

**Practice**: Timeboxed focused work sessions with clear objectives and proper handoffs.

**Structure**:
- **Duration**: 60-90 minutes of focused work
- **Objective**: Single clear goal per session
- **Documentation**: Session notes created while context is fresh
- **Handoffs**: Explicit pause points with state documentation
- **Recovery**: Git commits enable easy recovery between sessions

**Why It Works**:
- Maintains focus on specific problems
- Prevents burnout on complex issues
- Creates natural break points for reflection
- Enables asynchronous work patterns
- Forces articulation of current state

**Example from Phase 8**:
Session 2 achieved a +55pp accuracy improvement in 45 minutes by focusing exclusively on document type boosting. Clear objective: "Fix document authority problem." Result: ADRs ranked higher than session notes through calculated 1.5× boost factor.

**Session Artifacts**:
- Starter document (objectives, context, approach)
- Detailed session notes (decisions, experiments, results)
- Handoff summary (next steps, open questions)
- Git commits (recovery points)

---

## 2. ADR Framework

**Practice**: Document architectural decisions with context, alternatives, and consequences.

**Structure**:
```markdown
## Context
What situation requires a decision?

## Options Considered
1. Option A: Pros, Cons, Fit
2. Option B: Pros, Cons, Fit
3. Option C: Pros, Cons, Fit

## Decision
Chosen option with rationale

## Consequences
Positive, Neutral, Negative impacts

## Alternatives Rejected
Why other options weren't chosen (with data)
```

**Why It Works**:
- Captures reasoning at decision time
- Documents alternatives considered
- Enables future evaluation of decisions
- Creates searchable knowledge base
- Prevents revisiting settled questions

**Example from Phase 8**:
ADR-009 documented the decision to use hybrid search (BM25 + Vector) rather than vector-only search. Alternatives like re-ranking models and Small-to-Big retrieval were explicitly rejected with data showing why they weren't suitable. When chunk size assumptions were validated (1847 chars vs expected 186 chars), Small-to-Big was abandoned without wasted implementation effort.

**Decision Criteria**:
- Weighted scoring when quantifiable
- Explicit trade-offs documented
- Data-driven when possible
- Rollback plans considered

---

## 3. Validation-First Approach

**Practice**: Build measurement and test infrastructure before optimization.

**Principle**: You can't improve what you can't measure. Create test suites that define success before attempting solutions.

**Process**:
1. **Baseline**: Measure current state comprehensively
2. **Test Suite**: Create representative evaluation set
3. **Metrics**: Define quantifiable success criteria
4. **Validate**: Test before and after every change
5. **Iterate**: Use metrics to guide next steps

**Why It Works**:
- Prevents optimization without validation
- Creates objective success criteria
- Enables comparison of approaches
- Catches regressions immediately
- Guides systematic improvement

**Example from Phase 8**:
Session 0 created a 20-query test suite before any optimization. Queries covered factual (50%), architectural (25%), comparative (15%), and how-to (10%) categories. This test suite revealed 0% accuracy on factual queries, drove all subsequent improvements (0% → 20% → 75% → 90%), and provided objective validation of each optimization layer.

**Test Suite Design**:
- Representative query distribution
- Multiple difficulty levels
- Known correct answers
- Quantifiable pass/fail criteria
- Regular re-validation

---

## 4. Data-Driven Engineering

**Practice**: Make decisions based on measurement and analysis rather than assumptions.

**Approach**:
- **Diagnose First**: Build tools to understand current behavior
- **Measure Gaps**: Quantify the problem before solving
- **Calculate Solutions**: Derive parameters from data, don't guess
- **Validate Assumptions**: Test beliefs before committing
- **Iterate on Data**: Let measurements guide next steps

**Why It Works**:
- Eliminates guesswork
- Validates intuitions with evidence
- Finds root causes faster
- Prevents wasted effort on wrong problems
- Creates defensible engineering decisions

**Example from Phase 8**:

**Diagnostic Tools**: Built ranking analyzer to understand why wrong documents appeared. Discovered ADR-001 ranked #8 for "What is Nomad?" despite being the definitive answer.

**Gap Measurement**: Calculated that ADR-001 needed to overcome a 0.034 similarity gap to reach top-3. This quantified the problem precisely.

**Calculated Solution**: Derived 1.5× boost factor from gap analysis rather than trying random multipliers. First attempt worked because it was calculated, not guessed.

**Assumption Validation**: Tested belief that chunks were too small (186 chars). Discovered actual chunks were 1847 chars. This 10× difference invalidated the Small-to-Big optimization plan, saving 4+ hours of wasted implementation. Pivoted to Topic Authority instead.

**Metrics Over Intuition**: Every optimization decision backed by test suite results. Session 2's document boosting improved factual accuracy from 20% to 75% (+55pp) with measurable evidence.

---

## 5. Git Workflow Discipline

**Practice**: Frequent commits with descriptive messages capturing both technical and business context.

**Commit Strategy**:
- **Frequency**: Commit after each meaningful change
- **Messages**: Technical change + business rationale
- **Recovery**: Small commits enable easy rollback
- **History**: Clean progression tells the story
- **Branches**: Experiment isolation when appropriate

**Message Format**:
```
Component: Brief technical change

Business context explaining why this matters.
Impact on users/system/goals.
```

**Why It Works**:
- Provides recovery points during experimentation
- Creates searchable history of decisions
- Enables rollback of failed experiments
- Documents reasoning at commit time
- Facilitates collaboration and review

**Example from Phase 8**:
During optimization work, made small frequent commits after each test suite run. When document boosting worked, the commit message captured both technical change ("Add document type boosting with 1.5× ADR multiplier") and business impact ("Factual accuracy 20% → 75%, +55pp improvement in 45 minutes"). Zero work was lost despite multiple session interruptions because commits created natural recovery points.

**Commit Discipline**:
- Never lose more than 30 minutes of work
- Every commit builds/runs
- Messages explain "why" not just "what"
- Clean history aids future debugging

---

## 6. AI-Assisted Engineering

**Practice**: Leverage AI as collaborative partner throughout the development lifecycle, from initial planning through implementation to validation.

**Philosophy**: AI engagement begins at Session 0, not just validation. Use AI as thinking partner for architecture design, implementation support, and comprehensive review. Different AI models (Claude, Gemini) provide complementary perspectives.

### Planning & Architecture Design

**Practice**: Engage AI early in planning to validate approach and explore alternatives before coding.

**When to Use**:
- Session 0 strategy and architecture design
- Exploring unfamiliar technical domains
- Evaluating multiple architectural options
- Validating approach before implementation
- Understanding trade-offs and risks

**Example from Phase 8 Session 0**:
Before writing any optimization code, discussed the three-pillar strategy (Hybrid Search, Small-to-Big, Experiments) with Claude to validate the approach. AI helped:
- Confirm hybrid search was viable (BM25 + Vector feasible)
- Explore BM25 integration patterns and libraries
- Design test suite structure and query categories
- Identify potential pitfalls (chunking strategy risks)
- Get implementation guidance for rank-bm25 library

**Why It Works**:
- Surfaces architectural issues before time investment
- Explores alternatives faster than solo research
- Provides implementation patterns for unfamiliar tech
- Validates assumptions about feasibility
- Reduces thrashing during implementation
- Creates shared understanding of approach

**AI as Thinking Partner**: Not "validate my completed work" but "help me think through this problem." Architecture discussions, trade-off analysis, risk identification, and implementation strategy.

### Implementation Support

**Practice**: Use AI during implementation for pattern research, code examples, and technical clarification.

**When to Use**:
- Implementing unfamiliar libraries or patterns
- Debugging confusing error messages
- Optimizing code performance
- Understanding API documentation
- Exploring implementation alternatives

**Example from Phase 8**:
- **BM25 Integration**: Asked Claude for rank-bm25 usage patterns, got working example code
- **Document Boosting**: Explored score manipulation patterns with AI before implementing
- **Topic Authority**: Researched title-based matching approaches with Gemini
- **Error Resolution**: AI helped interpret confusing embedding dimension mismatches

**Why It Works**:
- Accelerates learning on unfamiliar topics
- Provides working code examples to adapt
- Explains error messages in context
- Suggests optimization approaches
- Reduces time stuck on implementation details

### Review & Validation

**Practice**: Share work with AI for comprehensive review before deployment.

**When to Use**:
- After implementing new functionality
- Before deploying to production
- When debugging complex issues
- To catch edge cases
- For architecture review

**Example from Phase 8**:

**Session 3 Validation**: Shared boost factor implementation with Google Gemini for architectural review. Gemini caught test suite bugs (incorrect expected results in 2 queries) and suggested refinements to boost factor calculation approach. Incorporating feedback prevented deploying buggy validation code.

**Alternative Exploration**: Used AI to explore Small-to-Big retrieval patterns and re-ranking approaches. Quickly evaluated feasibility before committing implementation time.

**Assumption Testing**: AI feedback helped validate hypothesis about chunk sizes before expensive investigation. When measurements contradicted assumptions (1847 chars vs expected 186), AI helped identify pivot to Topic Authority approach.

**Accelerated Debugging**: External validation helped identify edge cases in test queries that human review missed (queries with multiple valid answers).

**Why It Works**:
- Catches errors before deployment
- Provides alternative perspectives
- Identifies edge cases humans miss
- Validates architectural decisions
- Creates feedback loop for improvement

### Workflow Integration

**When to Engage AI**:

**During Planning** (Session 0):
- "Here's the problem and my approach - what am I missing?"
- "Compare these three architectural options"
- "What patterns exist for this problem?"
- "Help me design a test suite for this optimization"
- "What risks should I consider?"

**During Implementation** (Sessions 1-N):
- "How do I implement X in Python?"
- "What's the best library for Y?"
- "Explain this error message in context"
- "Optimize this code snippet"
- "Show me usage patterns for this library"

**When Stuck** (Mid-session):
- "Here's what I've tried, still not working - diagnostic ideas?"
- "Should I continue this approach or pivot?"
- "What am I not seeing in this data?"
- "Debug this confusing behavior"

**For Validation** (Session completion):
- "Review this implementation for bugs"
- "Does this architecture make sense?"
- "What edge cases am I missing?"
- "Critique this approach"
- "Verify my test suite covers key scenarios"

### Tool Selection: Claude vs Gemini

**Claude**:
- Implementation and code generation
- Real-time problem solving during sessions
- Pattern research and working examples
- Iterative refinement and debugging
- Quick technical clarification

**Gemini**:
- Architecture review and strategic validation
- Alternative perspective on approach
- Catching logical flaws and edge cases
- Higher-level design feedback
- Cross-validation of Claude suggestions

**Why Both**: Different models catch different issues. Cross-validation increases confidence. Gemini caught test suite bugs in Phase 8 that Claude missed. Claude provided implementation patterns Gemini couldn't generate as effectively.

**Best Practices**:
- Engage AI early (planning, not just validation)
- Be specific in requests (context + clear question)
- Iterate on feedback (refine based on AI responses)
- Cross-validate with multiple AIs (complementary perspectives)
- Document AI insights in session notes (capture reasoning)
- Use AI to accelerate, not replace, thinking

---

## Case Study: Phase 8 in Practice

**Challenge**: RAG system showing 0% accuracy on factual queries despite 100% accuracy baseline on simpler queries.

**Application of Methodology**:

### **Session-Based Development**
- Session 0: Built test suite and baseline (60 min)
- Session 1: Implemented BM25 hybrid search (60 min)
- Session 2: Added document type boosting (45 min - breakthrough!)
- Session 4: Fixed validation edge cases (60 min)
- Session 5: Validated chunk size assumptions (45 min)
- Session 6: Implemented topic authority (30 min - completion!)

Result: 7.5 hours across 8 sessions, clear progression, no burnout

### **Validation-First**
Created 20-query test suite before any optimization. Test suite drove all decisions:
- 0% baseline → quantified the problem
- 20% after hybrid search → showed progress
- 75% after document boosting → validated breakthrough
- 90% after topic authority → confirmed completion

### **Data-Driven Engineering**
- Measured gap: ADR-001 0.034 similarity points from top-3
- Calculated boost: 1.5× factor derived from gap analysis
- Validated assumptions: Chunks 1847 chars, not 186 chars
- Pivoted approach: Dropped Small-to-Big, added Topic Authority
- Every decision backed by test results

### **ADR Framework**
ADR-009 captured:
- Three-layer architecture decision (Hybrid + Boosting + Topic)
- Alternatives considered (re-ranking, Small-to-Big, fine-tuning)
- Why alternatives rejected (complexity, no value, wrong problem)
- Consequences (performance, maintenance, extensibility)

### **Git Discipline**
Small commits after each experiment:
- Easy rollback when boost factors too aggressive
- Clear progression visible in git log
- Business impact captured in commit messages
- Zero work lost across multiple days

### **AI-Assisted Engineering**
- Session 0: Validated three-pillar strategy with Claude before coding
- Session 1: Got BM25 integration patterns from AI (rank-bm25 library)
- Session 2: Explored score manipulation patterns with AI
- Session 3: Gemini caught test suite bugs (2 incorrect expected results)
- Session 5: AI helped interpret chunk size findings and pivot to Topic Authority
- Cross-validation: Claude for implementation, Gemini for architecture review

**Result**: 0% → 90% accuracy, exceeded 60-70% target, systematic and reproducible methodology.

---

## Methodology in Action

**When starting a new optimization**:
1. Create test suite (validation-first)
2. Measure baseline (data-driven)
3. Plan session objectives (session-based)
4. Implement with git commits (discipline)
5. Validate with external review (feedback)
6. Document in ADR (knowledge capture)

**When hitting blockers**:
1. Diagnose with tools (data-driven)
2. Measure the gap precisely
3. Seek external perspective (validation)
4. Consider alternatives (ADR framework)
5. Make small experiments (git safety net)
6. Validate each change (test suite)

**When completing work**:
1. Validate final state (test suite)
2. Document decisions (ADR)
3. Capture lessons learned (session notes)
4. Clean up git history (clear story)
5. Share for feedback (external validation)

---

## Tools & Practices

**Development**:
- Python for implementation (rapid iteration)
- pytest for validation (comprehensive testing)
- uv for environment management (reproducible setups)
- Git for version control (disciplined commits)

**Planning**:
- Session starter documents (clear objectives)
- ADR templates (structured decisions)
- Test suite definitions (quantifiable success)
- Markdown documentation (lightweight, versionable)

**Validation**:
- Test suites (objective measurement)
- External AI review (catch errors early)
- Metrics dashboards (continuous monitoring)
- Git diffs (change verification)

**Collaboration**:
- ADRs for decisions (asynchronous context)
- Session notes for progress (clear handoffs)
- Commit messages for history (searchable rationale)
- Documentation for knowledge (team enablement)

---

## Continuous Improvement

**Retrospectives**:
- After each phase: What worked? What didn't?
- Methodology evolution based on experience
- Pattern recognition across projects
- Tool refinement based on pain points

**Lessons from Phase 8**:
- Validation-first prevented premature optimization
- Data-driven approach found root causes faster
- Session-based structure prevented burnout
- AI-assisted engineering caught bugs early and accelerated implementation
- Git discipline enabled fearless experimentation
- ADRs captured context that would've been lost

**Future Refinements**:
- Expand test coverage earlier
- Build diagnostic tools proactively
- Seek validation more frequently
- Document assumptions explicitly
- Measure more, assume less

---

## Key Takeaways

**What makes this methodology effective**:
1. **Systematic over heroic**: Reproducible process beats lucky breakthrough
2. **Data over intuition**: Measurements reveal truth faster than guessing
3. **Validation over confidence**: Test suites catch errors humans miss
4. **Documentation over memory**: Write it down while context is fresh
5. **Iteration over perfection**: Small validated steps beat big leaps
6. **External over isolated**: Feedback prevents expensive mistakes

**When to apply**:
- Complex optimization problems
- Unfamiliar technical domains
- High-stakes architectural decisions
- Multi-step system improvements
- Learning new technologies
- Production system changes

**Expected outcomes**:
- Faster time to working solution
- Fewer dead ends and wasted effort
- Reproducible results
- Clear documentation trail
- Defensible engineering decisions
- Continuous improvement culture

---

## Interview Application

**Question: "How do you approach optimization?"**

Walk through validation-first approach:
1. Build test suite before optimizing
2. Measure baseline and gaps
3. Make data-driven changes
4. Validate each improvement
5. Document decisions in ADRs

**Question: "Tell me about debugging a complex issue"**

Explain data-driven diagnostic process:
1. Build tools to understand behavior
2. Measure gaps precisely
3. Validate assumptions with data
4. Iterate based on measurements
5. External validation catches errors

**Question: "How do you manage projects?"**

Describe session-based development:
1. Timeboxed focused sessions
2. Clear objectives per session
3. Frequent git commits
4. Documentation while fresh
5. Systematic progress over time

**Question: "How do you use AI in your development workflow?"**

Explain comprehensive AI integration:
1. Planning: Validate architecture in Session 0 with AI
2. Implementation: Get patterns and examples during coding
3. Debugging: Use AI to interpret errors and suggest fixes
4. Validation: Cross-validate with multiple AIs (Claude + Gemini)
5. Learning: Accelerate understanding of unfamiliar tech

**Example**: Phase 8 Session 0 - validated three-pillar strategy with Claude before writing code, got BM25 integration patterns, designed test suite structure. Session 3 - Gemini caught bugs Claude missed. Not just "ChatGPT for code" but systematic AI collaboration throughout lifecycle.

---

**Status**: Complete engineering methodology based on Phase 8 patterns  
**Application**: Optimization, debugging, architectural decisions, learning  
**Evidence**: Phase 8 journey (0% → 90% in 7.5 hours through systematic approach)
