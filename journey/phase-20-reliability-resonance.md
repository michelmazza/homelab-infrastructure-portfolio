# Phase 20: Reliability & Resonance

**The AIOps system now maintains the diagnostic thread over time.**

## The Arc

Phase 18 gave the system prediction — OLS regression over live Prometheus metrics,
time-to-threshold forecasting, observer effect detection. Phase 19 gave it
explanation — correlating metric inflections with infrastructure events to answer
"why is CPU high?" in under 10ms, zero LLM involvement.

Phase 20 closed the loop: memory and verification.

Ask "Why is CPU high?" and the system answers. Ask "Is that still related to the
restart?" ninety seconds later and the system knows what you're referring to,
checks the live trend, and tells you whether the original diagnosis still holds.

That's the arc:
- Phase 18: Predict
- Phase 19: Explain
- Phase 20: Remember and Verify

---

## The Engineering Problem

The system after Phase 19 was stateless at the conversation level. Each query
was a fresh start. Diagnostic context was computed, displayed in the UI, then
discarded. The follow-up capability — the conversational diagnostic thread
that makes an observability system genuinely useful — didn't exist.

Three questions had no answer:
1. How do you persist a causal attribution across conversation turns without
   contaminating the LLM's context window with raw infrastructure data?
2. How do you distinguish a genuine follow-up question from a new diagnostic
   query that happens to use follow-up phrasing?
3. How do you prevent a system from serving a cached conclusion that was true
   five minutes ago but may no longer be?

Phase 20 answered all three.

---

## Three-Tier Structure

**Tier 1 — Debt Clearance**: Before any new capability, clear the accumulated
technical debt. Fix test infrastructure. Migrate the Prometheus scraper from a
background process to a proper cluster-managed job. Arrive at a verified,
honest baseline.

**Tier 2 — Reliability**: Harden the observability foundation. A heartbeat with
semantic content (not just a timestamp) that distinguishes "collector running,
infrastructure quiet" from "collector down." A unified retention manager that
coordinates time-based pruning across both data tables in a single atomic
transaction. WAL mode consistency across all database connections. Scheduled
daily operations managed by the cluster scheduler.

**Tier 3 — Resonance**: The conversational layer. Structured diagnostic context
that survives turns. A context window guard that caps every diagnostic summary
at 200 characters — the only sanctioned path for diagnostic data to reach the
language model. Follow-up detection with a new-query override. Verifiable
Inference. The Diagnostic Thread timeline in Tab 8.

---

## The Key Innovations

### Heartbeat Semantic Contract

The previous gap detection used the presence of any recent log event as a proxy
for collector health. A production system with no recent deployments produces no
events — and would trigger a false gap warning.

The solution: a heartbeat inserted after every collection pass, with a detail
field encoding the result. "ok" means both infrastructure APIs responded
successfully. Error codes distinguish API degradation from true collector failure.
Only an "ok" heartbeat proves the infrastructure is quiet rather than the
collector being down.

This resolves what Gemini called the "Quiet Infrastructure Ambiguity" — a subtle
but important correctness property for any observability system.

### DiagnosticContext and the Context Window Guard

Diagnostic context is structured as a dataclass with seven fields. The contract
is strict: the only path by which a diagnostic conclusion reaches the language
model's context window passes through a single bounded function that caps output
at 200 characters. No raw fields. No unbounded injections.

Dual-layer truncation enforces this invariant: field-level truncation on the
inputs (top cause, node name), then a string-level hard cap on the output.
Future template changes cannot silently violate the 200-character contract.

The TTL is extracted to configuration with an environment variable override —
what Gemini called "Observability Tempo": different deployments may need different
context validity windows.

### Follow-up Detection with New-Query Override

Twelve follow-up phrases are recognized. One override: if a query contains
both a follow-up phrase and the signals for a new diagnostic query (metric
reference plus spike signal), the new diagnostic path takes priority.

The reasoning: stale context is the worst outcome. A user who says "why is
memory spiking now?" wants fresh correlation, not a cached answer about CPU
from three minutes ago — even if their phrasing superficially resembles a
follow-up.

### Verifiable Inference

When a follow-up response serves cached context, it appends the current live
trend: "Current status: INCREASING as of 12 seconds ago." The live check runs
at follow-up time, not at cache time. If the live check fails for any reason,
the base summary returns without it — graceful degradation is a hard requirement.

This prevents what Gemini called the "Stateless Oracle" failure mode: a system
that recites historical data as if it were current truth.

### Tab 8: Diagnostic Thread

An append-only session investigation timeline. Every query that produces a
causal attribution adds an entry: timestamp, metric, top cause, inflection type,
confidence. Rendered reverse-chronologically. The Pivot Event — when a user
switches from investigating CPU to investigating memory — appears naturally as
two entries in the timeline.

Gemini called this the portfolio anchor: "It doesn't just show the result;
it shows the diagnostic journey. Recruiters and engineers look for how a
system thinks."

---

## Results

- 999 automated tests, 980/980 runnable passing at 100%
- Zero regressions across all 9 sessions
- 3 new modules: observability lifecycle manager, diagnostic context, configuration
- 9 Gemini mandates addressed (4 from Checkpoint 1, 5 from Checkpoint 2)
- Zero-LLM analytical streak: 5 consecutive phases (Phases 16–20)
- 4-node cluster at the time of this phase: 3 Proxmox VMs plus the local ML workstation as a proper cluster node (the workstation was later removed from the compute plane — the cluster is 3 nodes today, and the workstation serves GPU inference and image builds only)
- Grade: A+ (Gemini Checkpoint 3)

---

## What Gemini Said

> "The completion of Phase 20 marks a definitive shift from building a 'tool'
> to engineering a 'system.' By integrating deterministic reliability with
> conversational state, the project has moved into a rare category of
> production-grade observability."

> "Reaching 999 tests with zero regressions while maintaining a 5-phase
> Zero-LLM analytical streak is a masterclass in disciplined development."

---

## The Recruiter Demo

Ask it "Why is CPU high?" It correlates metric inflections against infrastructure
events and responds in under 10ms — no LLM in the analytical path. Then ask
"Is it still related to that restart?" It remembers the diagnosis, checks the
live trend, and tells you whether the original conclusion still holds.

Not because it's calling a language model for each answer. Because it maintains
a per-incident diagnostic context with bounded TTL, context-guarded injection,
and live infrastructure verification.

That's the Conversational AIOps arc: predict, explain, remember, verify.

---

**Phase**: 20 — Reliability & Resonance
**Grade**: A+ (External Validation)
**Tests**: 999 total, 980/980 runnable @ 100%
**Zero-LLM Streak**: 5 consecutive phases (16–20)
**Counterpart**: Phase-20-Journey-Narrative-PRIVATE.md (full technical detail, private repo)
