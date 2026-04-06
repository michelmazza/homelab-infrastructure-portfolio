# ADR-016: Diagnostic Context Architecture — Bounded State for Conversational AIOps

**Status**: Accepted
**Date**: 2026-04-06
**Phase**: 20 — Reliability & Resonance
**Grade**: A+ (Gemini CP1 + CP2 + CP3)

**Counterpart**: Private repository `ADR-016-Diagnostic-Context-Architecture.md`
(full technical detail including hostnames, connection strings, and implementation code)

---

## Context

Phase 19 built a correlation engine that answers "why is CPU high?" by correlating
metric inflections against infrastructure events. The result — a causal attribution
(entity, event_type, score, inflection_type) — was displayed in the Glass Box UI
but not persisted across conversation turns.

This created the **Stateless Oracle Problem**: the system computed accurate
diagnostic conclusions but had no memory of having done so. Follow-up questions
("Is that still the case?") started from zero. Cached conclusions could become
stale without detection.

Phase 20 required a design for conversational diagnostic persistence satisfying
four constraints simultaneously:

1. **Context window safety**: Diagnostic data injected into the LLM must be
   bounded — unbounded infrastructure data produces inconsistent responses.
2. **Testability without UI coupling**: Persistence logic must be testable
   without mocking the Streamlit session state layer.
3. **Stale context prevention**: The system must distinguish a valid cached
   conclusion from one that has expired. Serving stale context is worse than
   no context.
4. **New query priority**: A follow-up phrase in a query that also contains new
   diagnostic signals must trigger fresh correlation, not serve cached context.

---

## Decision

Implement `DiagnosticContext` as a structured dataclass with a bounded summary
function and TTL-based staleness check, following a strict layered architecture:

```
correlate() → DiagnosticContext (structured)
               ↓
         is_context_stale() (TTL check)
               ↓
         diagnostic_context_to_summary() (context window guard)
               ↓
            LLM prompt (≤200 chars injected)
```

No other code path may inject diagnostic context into the LLM.

---

## DiagnosticContext Structure

Seven fields capture a complete causal attribution:

| Field | Type | Description |
|-------|------|-------------|
| `metric` | str | Full composite key (metric + node identifier) |
| `node` | str | Infrastructure node identifier |
| `inflection_type` | str | "step_change" or "trend_shift" |
| `top_cause` | str | Causal event label |
| `top_cause_score` | float | Multiplicative score (0.0–1.0) |
| `event_type` | str | Infrastructure event type |
| `timestamp` | float | Unix epoch — when correlation was computed |

---

## Context Window Guard

`diagnostic_context_to_summary()` is the **only sanctioned path** for diagnostic
context to reach the LLM's context window. Contract: output ≤200 characters.

**Dual-layer truncation** enforces this invariant:
- **Layer 1 (field-level)**: Truncate `top_cause` and `node` inputs before template construction
- **Layer 2 (string-level)**: Hard cap on the final string as a safety net

This dual-layer design guarantees the invariant regardless of future template
changes. A single-layer approach would be silently violated if the template grew.

**Result**: 8 tests validate the contract — zero truncation violations in all
production smoke tests.

---

## TTL and Observability Tempo

`is_context_stale()` checks elapsed time against a configurable TTL (default 300s).

The TTL is extracted to a configuration module with environment variable override,
enabling per-deployment tuning without code changes. Gemini named this the
**Human Deliberation Buffer**: the window within which an operator is likely
still investigating the same incident. Different infrastructure environments
may need different values.

---

## Follow-up Detection and New-Query Override

Twelve follow-up phrases are recognized ("tell me more", "is it still", "what
caused it", etc.). One critical override: if a query matches a follow-up phrase
AND contains the signals for a new diagnostic query (metric reference + spike
signal), the new diagnostic path takes priority.

**Rationale**: Stale context is the worst outcome. A user who can formulate
"why is memory spiking now?" wants fresh correlation — not a cached CPU diagnosis
from three minutes ago, regardless of how their phrasing reads.

---

## Context Persistence: Return, Not Side-Effect

The correlation suffix builder was refactored from a side-effecting function
to one that returns a `(suffix, DiagnosticContext | None)` tuple. The caller
stores context in session state after receiving the tuple.

**Why this matters**: Pure function → fully testable without mocking the UI
layer. The refactor enabled the complete follow-up test suite without any
Streamlit dependency injection.

---

## Verifiable Inference

When the follow-up path serves cached context, it appends live trend status:
`"Current status: INCREASING as of 12s ago."` The live check runs at follow-up
time, not at cache time.

Key implementation detail: the composite metric key stored in `DiagnosticContext`
must be split before calling the trend monitor, which expects metric base
and node as separate arguments. `rsplit("_", 1)` (split from the right, once)
handles multi-underscore identifiers correctly.

Failure handling: the live check failure never blocks a follow-up response.
Graceful degradation is a hard requirement.

---

## Diagnostic Thread (Tab 8)

An append-only list in session state captures every `correlate()` candidate
result across a session. Tab 8 renders this as a reverse-chronological
investigation timeline.

The **Pivot Event** (Gemini's term for switching from one metric investigation
to another) is handled naturally: each candidate appends a new entry regardless
of what preceded it. No special pivot-detection logic is needed.

---

## Rejected Alternatives

### Raw correlation results in session state
**Rejected**: Unbounded data. The context window guard cannot be enforced on
raw event objects.

### LLM-generated summary at correlation time
**Rejected**: Adds LLM call latency to the correlation path and produces
non-deterministic output (untestable). The deterministic bounded function is
zero-latency and fully unit-testable.

### Database persistence for context
**Rejected**: Session state is the correct scope — per-session, per-user,
in-process. Database persistence is for cross-session memory (ADR-014 pattern).

### Context valid indefinitely
**Rejected**: The Stateless Oracle failure mode reappears. A stale diagnostic
conclusion served as current truth is operationally dangerous.

### Single-layer string truncation only
**Rejected**: Template boilerplate is ~170 chars. Future template changes could
silently push output over 200 chars. Dual-layer guarantees the invariant.

---

## Results

| Metric | Result |
|--------|--------|
| Context window invariant | ≤200 chars — guaranteed by dual-layer truncation |
| TTL enforcement | 300s default, env-var configurable |
| Follow-up detection | 12 phrases + new-query override |
| Verifiable Inference | Live trend appended on follow-up when available |
| Graceful degradation | Live check failure never blocks follow-up |
| Test coverage | 12 tests across 3 files, all passing |
| Grade | A+ (Gemini CP1 + CP2 + CP3) |

---

## Consequences

**Positive**:
- Context window injection is bounded and contract-enforced
- TTL prevents stale conclusions from persisting across incidents
- Pure function design enables full test coverage without UI mocking
- Verifiable Inference prevents "oracle recitation" of historical data as current truth
- Diagnostic Thread provides session-scoped investigation audit trail

**Accepted tradeoffs**:
- `diagnostic_context` is a single-entry cache: only the most recent candidate
  is available for follow-up. The full session history lives in `diagnostic_thread`.
- TTL=300s is a heuristic. Tunable via env var for different deployment contexts.

---

## External Validation

Gemini coined five portfolio-grade concepts in the process of validating this design:

| Concept | Maps To |
|---------|---------|
| **Verifiable Inference** | Live trend check appended to follow-up responses |
| **Diagnostic Thread** | Append-only session investigation timeline (Tab 8) |
| **Pivot Event** | Natural transition between diagnostic investigations |
| **Human Deliberation Buffer** | TTL=300s — operator's active incident window |
| **Observability Tempo** | Configurable TTL as deployment-specific tuning parameter |

> "The completion of Phase 20 marks a definitive shift from building a 'tool'
> to engineering a 'system.'"
>
> — Gemini Checkpoint 3

---

## Related Decisions

- **ADR-014** (Phase 15): Conversation memory foundation — session lifecycle, SQLite storage
- **ADR-015** (Phase 16): Analytical intelligence — metric snapshots, anomaly detection
- **Phase 19 Correlation Engine**: `correlate()` produces the DiagnosticContext this ADR governs

---

**ADR**: 016
**Phase**: 20 — Reliability & Resonance
**Status**: Accepted
**Grade**: A+ (External Validation)
**Created**: 2026-04-06
**Portfolio Version**: This document (sanitized)
