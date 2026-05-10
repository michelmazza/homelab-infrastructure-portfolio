# ADR-025: 5-Gate Live Execution Pipeline — Deterministic Autonomous Remediation

**Status**: Accepted
**Date**: 2026-04-26
**Phase**: 23 — The Autonomous Bridge
**Grade**: A+ (Gemini CP1 Week 1 + CP2 Week 2)

**Counterpart**: `ADR-025-Live-Execution-Pipeline.md` in this repository
(full architectural detail, real module paths, real timing constants,
code snippets).

---

## Context

For seven consecutive phases, the Conversational AIOps system had operated as
an advisory observer. By Phase 22 it could correlate metric inflections with
infrastructure events, suggest remediation actions from a version-controlled
allowlist, and simulate those actions through a three-gate dry-run safety
architecture (kill-switch, rate-limit, loop guard). It could *tell* an operator
what to do, and it could *pretend* to do it.

Phase 23 needed to bridge from advisory (Level 1) to autonomous remediation
(Level 2). The naive shape of that bridge was: "add an LLM agent that decides
what action to execute." This is the wrong shape of risk for production
infrastructure. LLMs hallucinate action classes. LLMs return malformed plans.
LLMs fail in ways that are hard to enumerate and harder to test. An LLM in
the safety-critical execution path is a liability without commensurate value.

The required design satisfied four constraints simultaneously:

1. **Determinism**: Every gate decision must be reproducible from a fixed
   set of inputs. No probabilistic reasoning in the execution path.
2. **Single failure-mode rollback**: If the system acts and the underlying
   problem persists, the system must autonomously undo the action — without
   waiting for additional failures to "confirm" the decision.
3. **Structured observability**: Every execution attempt must produce a
   timing-precise, queryable audit record. No silent state transitions.
4. **Independent abort paths**: At least three independent safety controls
   must be able to halt execution without coordination — pattern-level,
   global, and post-action.

---

## Decision

Implement `execute_live(alert, pattern, dry_run=False)` as a deterministic
state machine with five gates. Each gate is independently testable,
independently observable, and independently abortable.

```
   ┌──────────────────────────────────────────────────────────┐
   │                 execute_live(alert, pattern)             │
   └──────────────────────────────────────────────────────────┘

   [1] Pre-flight ──→ failure events in 60s lookback?
          │              ├─ yes: ABORT (alert is stale)
          │              └─ no:  proceed
          ▼
   [2] Action ──→ invoke remediation API
          │              ├─ error: ABORT (record failure)
          │              └─ ok:    proceed
          ▼
   [3] Verify ──→ confirm action took effect
          │              ├─ no:  ABORT (record failure)
          │              └─ yes: proceed
          ▼
   [4] Stable-duration ──→ poll for 60s
          │              ├─ any failure: TRIGGER ROLLBACK
          │              └─ stable:      proceed
          ▼
   [5] Record ──→ persist outcome with full timing
```

No LLM is invoked at any gate. All thresholds are static and configuration-
controlled. All checks are deterministic queries against the structured event
log, the orchestrator's allocation API, or the local outcome database.

---

## Gate Specifications

### Gate 1 — Pre-flight

Queries the structured event log for any of **eleven failure event types**
within a **60-second lookback window**. Failure types include allocation
failures, OOM-kills, health-check failures, and similar terminal-state
transitions. If any are present, the pipeline aborts before action.

**Why this gate exists**: The alert that triggered this execution may have
been generated minutes ago. By the time the pipeline reaches Gate 1, the
underlying problem may have already escalated past the point where the
proposed action is helpful — or the system may have already self-recovered.
Pre-flight ensures the action is still actionable.

### Gate 2 — Action

Invokes the orchestrator API for the pattern's declared action class.
Phase 23 shipped two action classes: a restart-class action (existing
pattern, validated in dry-run since Phase 22) and a script-class action
(`disk_cleanup`, demonstrating extensibility beyond a single action type).

**Why two action classes**: A second restart-class pattern would have proved
nothing the first didn't. The script-class pattern proved the pipeline can
execute fundamentally different action types within the same five-gate
structure. This is the foundation for a broader pattern library.

### Gate 3 — Verify

Confirms the action took effect by polling the orchestrator's allocation
status API. Distinguishes "action submitted" from "action observed in
infrastructure state."

**Why this gate exists**: API success ≠ action success. The orchestrator
may accept a job submission and queue it; the actual restart may fail at
placement. Verify closes that gap.

### Gate 4 — Stable-Duration

Polls for **60 seconds** after Verify succeeds. Triggers rollback on **any
single failure event** observed during the window.

**Why a single failure triggers rollback**: Production self-healing systems
that wait for "confirmation" failures (two strikes, three strikes) compound
the original problem. If the action made things worse, the system needs to
know within seconds, not minutes.

The rollback path uses **ExternalID + CreateTime** matching to ensure the
system tracks the same allocation it acted on. A 2-second drift tolerance
on CreateTime accommodates clock skew without admitting unrelated
allocations into the rollback decision.

### Gate 5 — Record

Persists a `pattern_outcome` row with **nanosecond-precision timing**, the
final outcome state (success / failed / rolled-back / in-progress), the
pattern identifier, and structured links to the originating alert. The
audit trail summary row in the operator UI surfaces the most recent
outcome with color-coded status.

**Why nanosecond precision**: Causal analysis after an incident requires
ordering events that may occur within the same millisecond. Coarser
timestamps obscure causation.

---

## Three Independent Safety Controls

The pipeline operates within three independent safety controls. Each can
abort execution without coordination with the others.

| Control | Scope | Mechanism | When Evaluated |
|---------|-------|-----------|----------------|
| **Pattern-level `live_enabled` flag** | Per-pattern | YAML allowlist field | Before alert routing |
| **Global kill-switch** | All patterns | CLI + UI, persisted state | Before alert routing |
| **Stable-duration rollback** | Per-execution | 60s post-action poll | After Gates 1–3 succeed |

Pattern-level is the finest-grained control: a new pattern defaults to
`live_enabled: false` until explicitly promoted. Global kill-switch is
the operator's emergency brake (see ADR-026). Stable-duration is the
post-action net that catches actions that were authorized but went wrong.

**Defense in depth**: A failure in any one control does not compromise
the others. A misconfigured pattern flag does not bypass the kill-switch.
A kill-switch toggle does not prevent rollback on already-running
executions.

---

## Rejected Alternatives

### LLM as decision-maker in the execution path

**Rejected**: Hallucination, malformed plans, and non-determinism are
unacceptable in a path that mutates production infrastructure state. The
deterministic pipeline is not a permanent rejection of LLM integration —
the next phase introduces LLM as **planner** that proposes actions which
the deterministic pipeline disposes. Cognition and execution are separated.

### Single-gate "execute and observe"

**Rejected**: Collapsing the five gates into "do the action, watch what
happens" yields a system with no defined safety contract. No way to
distinguish stale alerts (Gate 1) from API failures (Gate 2) from
placement failures (Gate 3) from post-action regressions (Gate 4). Each
failure mode requires distinct handling; conflating them obscures the
diagnostic story.

### Confirmation-failure rollback (require N failures before reverting)

**Rejected**: A system that waits for the second strike has already
extended the incident. If the action made things worse, the operator
needs the system to know that as soon as the first failure event arrives.
Single-failure rollback is the right asymmetry for autonomous remediation:
err on the side of reverting.

### LLM-generated audit narratives

**Rejected**: The audit trail is queried by deterministic UI filters and
serves as evidence for incident reviews. LLM-generated narratives introduce
non-determinism and add latency to the Record gate. Structured fields with
human-readable labels serve the same purpose without those costs.

### Indefinite stable-duration window

**Rejected**: A 60-second window is a heuristic, but an unbounded window
holds resources indefinitely on systems where the action did succeed.
60 seconds is long enough to catch immediate post-action regressions and
short enough to clear the pipeline for the next alert. Tunable per
deployment if the heuristic proves wrong for a given infrastructure.

---

## Results

| Metric | Result |
|--------|--------|
| Pipeline gates | 5 (pre-flight, action, verify, stable-duration, record) |
| Action classes shipped | 2 (restart-class, script-class) |
| Independent safety controls | 3 (pattern flag, kill-switch, rollback) |
| Pre-flight failure event types | 11 |
| Pre-flight lookback window | 60 seconds |
| Stable-duration window | 60 seconds |
| Rollback trigger threshold | 1 failure event (single-strike) |
| Outcome timing precision | Nanosecond |
| LLM calls in execution path | 0 |
| Tests added (Phase 23) | 27 (1154 → 1181 collected) |
| Phases without LLM in execution path | 8 consecutive (Phases 16–23) |
| Phases without test regression | 8 consecutive (Phases 16–23) |
| External grade | A+ (CP1 Week 1) + A+ (CP2 Week 2) |

---

## Consequences

**Positive**:

- Deterministic execution path is fully testable without LLM mocking.
  Every gate has a clear input contract and a clear output contract.
- Single-strike rollback contains incident scope. The system either
  succeeds within 60 seconds or reverts within 60 seconds.
- Three independent safety controls provide defense in depth without
  coordination overhead.
- Two action classes (restart, script) prove the pipeline generalizes
  beyond a single remediation type.
- Nanosecond-precision audit trail supports post-incident causal analysis
  at the resolution where it actually matters.
- Architecture admits LLM integration as **planner** in the next phase
  without disturbing the execution contract.

**Accepted tradeoffs**:

- The pattern allowlist must be maintained in version control. New
  remediation actions require an explicit pattern definition, code review,
  and dry-run validation period before live promotion.
- Single-strike rollback may revert successful actions if a coincident
  unrelated failure occurs in the 60-second window. The cost of
  occasional false-revert is judged lower than the cost of letting a
  bad action persist.
- 60-second windows on Gates 1 and 4 are heuristics. Tunable via
  configuration when deployment evidence justifies it.
- Pre-flight uses an allowlist of known failure event types. New failure
  modes from upstream tools require allowlist updates to be considered
  by Gate 1.

---

## External Validation

> "We built a deterministic, safe, and verifiable autonomous remediation
> system that operates with nanosecond precision and human-in-the-loop
> overrides."
>
> — Gemini CP2 (Week 2), narrative framing adopted directly into the
> Phase 23 Completion Report

Gemini CP1 (Week 1) cited the pre-flight gate's failure-event typing as
**"production-grade defensive programming"** — a phrasing that captures
the design intent. Pre-flight is not a "nice to have" check. It is the
gate that prevents the system from acting on alerts that have already
been overtaken by events.

---

## Related Decisions

- **ADR-026** (Phase 23): Kill-Switch with Confirmation Gate — the global
  safety control that operates in series with the pipeline
- **ADR-027** (Phase 23): Three-Layer UI Validation — the testing
  methodology that emerged from kill-switch UI bugs and now governs all
  UI features
- **ADR-016** (Phase 20): Diagnostic Context Architecture — the
  conversational state foundation Phase 23 builds on
- **Phase 22 Three-Gate Dry-Run Architecture**: The Level 1 safety
  pattern (kill-switch, rate-limit, loop guard) that the 5-gate pipeline
  layers execution onto without rewriting

---

## Forward Carry — LLM as Planner

The next phase introduces an LLM as a **planner** that generates action
plans. The 5-gate pipeline is preserved unchanged. The LLM proposes;
the pipeline disposes. LLM failure modes (hallucination, wrong action
class, malformed plan) are caught at the pipeline boundary — specifically
at the YAML allowlist match step before Gate 1, where any action not
matching a defined pattern is rejected.

This is the architectural payoff of separating cognition from execution:
the safety contract does not change when the cognitive layer changes.

---

**ADR**: 025
**Phase**: 23 — The Autonomous Bridge
**Status**: Accepted
**Grade**: A+ (External Validation, two consecutive checkpoints)
**Created**: 2026-04-26
**Portfolio Version**: This document (sanitized for public repository)
**Private Counterpart**: Phase 23 Completion Report §5.1, §6.2 + agent_actions module
