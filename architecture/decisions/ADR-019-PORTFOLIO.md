# ADR-019: Alerting Integration — Webhook Enrichment Pattern

**Status**: Accepted
**Date**: 2026-04-12
**Phase**: 21 — Distributed Foundations
**Deciders**: Michel Mazza

---

## Context

Earlier phases built a zero-LLM analytical stack: predictive regression,
correlation scoring, persistent diagnostic context, follow-up
detection, and an investigation-timeline UI. The system could answer
"why is this metric spiking?" in its own dashboard with a full Top
Cause reasoning trace.

But the reasoning stayed inside the UI. An operator receiving an
alert notification at 3am saw only the alert name and the raw metric
value — the same payload the monitoring system would emit without
this project existing. The Top Cause output had no path into the
operator's alerting workflow.

This was the **passive observer** posture: the system knew the
answer but did not offer it at the point of need. External feedback
flagged this gap as an obvious next step:

> "Surfacing diagnostics in alert annotations completes the
> self-healing story. It transforms the system from a passive
> observer into an active participant in infrastructure health."

---

## Decision

**Implement alert enrichment as a pure function: take an incoming
alert payload, call the correlation engine for Top Cause, append the
result as an annotation, return the enriched payload. Wire it via the
standard webhook receiver pattern when an HTTP surface is added in a
future phase.**

### Why Webhook Enrichment, Not Direct Injection

Three alternatives were considered and rejected, each for different
reasons.

**Direct annotation injection into the monitoring system.** Monitoring
systems like Prometheus keep alert annotations static (defined at
configuration time) and time-series (stored with the alert state).
There is no API to inject dynamic annotations from an external
service. This would have required modifying the monitoring system
itself.

**Calling the alerting router's API.** Alertmanager-style routers
generally do not expose an annotation-injection API — they accept
alerts from the monitoring system and route them to receivers. You
cannot call the router from outside to "add an annotation to alert X."

**Full HTTP webhook server in this phase.** The enrichment function
itself is pure logic: take a payload, extract metric and node, call
the correlation engine, append annotation, return enriched payload.
Wrapping it in an HTTP server adds a deployment surface (port,
health check, restart policy, orchestration manifest) for no
incremental value on the architectural story. External feedback
validated this: *"Logic-only is sufficient for this phase. Adding an
HTTP wrapper is a trivial Day 2 task."*

### The Data Flow

```
Monitoring system alert fires
        ↓
Alerting router receives
        ↓
Alerting router posts to webhook URL     (future: HTTP endpoint)
        ↓
enrich_alert(payload, db_url)           ← current scope (logic + tests)
        ↓
Correlation engine queries database
        ↓
Top Cause candidate returned
        ↓
Annotation appended to payload
        ↓
Enriched payload forwarded downstream    (future: chat, paging, email)
```

The scope of this ADR is the middle four steps — the pure logic,
tests, and documentation. The first two and last two are deferred.

### Graceful Degradation Is First-Class

Every failure mode returns the alert unchanged. The enrichment
function is never allowed to block a notification. The rationale: an
operator expecting an alert at 3am should not be deprived of the
alert because the enrichment pipeline has a bug. Alert enrichment is
*value-added*, not *load-bearing*.

This is enforced by a bare exception handler at the function
boundary. All internal functions may raise; the boundary swallows
everything and returns the alert as it came in.

### Why Logic-Only Is Sufficient

The hard part of alert enrichment is not the HTTP surface — it is
the *reasoning*. The monitoring system generates alerts; the router
forwards them; webhooks receive them. All three are standard,
well-documented technologies with many reference implementations.
Adding another HTTP server does not demonstrate engineering
judgment; it demonstrates operational busywork.

What demonstrates engineering judgment is:

1. The correlation engine can be *called* in a context other than
   the UI without modification — it is already decoupled.
2. The enrichment function gracefully degrades under six documented
   failure modes.
3. The annotation format is stable and readable in downstream
   notification channels.
4. The data flow respects the architectural constraint that no LLM
   sits on the critical path for alert enrichment.

All four are provable from the enrichment module and its test suite
without a running HTTP server. The HTTP server, when it exists, will
be boilerplate and an orchestration manifest. The interesting
engineering is in the logic.

---

## Consequences

### Positive

- **Top Cause reasoning reaches the operator at the point of need.**
  An operator receiving an alert at 3am sees "this is probably
  caused by X" in the same message, not in a dashboard they have to
  navigate to.
- **Zero-LLM analytical stack maintained.** Alert enrichment is
  regex metric extraction, a scored correlation lookup, and string
  formatting. No LLM calls on the critical path.
- **Reuses existing infrastructure.** The correlation engine is
  already the source of truth for Top Cause reasoning. This adds a
  second consumer (enrichment) with minimal new surface area.
- **Graceful degradation is a first-class concern.** Six documented
  failure modes, all returning the alert unchanged. The enrichment
  function cannot break an alert; it can only fail to add value.

### Negative

- **Not actually wired to the alerting router yet.** The function
  exists and has tests; the HTTP server to call it does not. An
  operator receiving an alert today does not yet see the enrichment.
  The work is proven-but-unreleased until the HTTP wrapper lands.
- **Metric extraction is convention-bound.** Works for three
  observed alert-name patterns. Any future alert that doesn't follow
  the convention needs either a new extraction path or manual
  mapping.
- **Single candidate only.** If an alert has multiple root causes,
  the operator sees only the top-scoring one. This can be misleading
  in cascading failures. Future work can lift this.
- **Annotation format is string-optimized, not JSON.** Downstream
  consumers that want structured data would need to parse the
  string. Acceptable for the current scope; may need a format option
  later.

### Neutral

- **No feedback loop yet.** The operator cannot tell the system
  "that Top Cause was wrong." Future UI work could add this.

---

## Test Coverage

24 tests in four groups:

- **Metric extraction** — 6 tests covering the three observed
  alertname patterns, malformed inputs, and missing fields.
- **Node extraction** — 4 tests covering the primary parse path and
  the label-based fallback.
- **Graceful degradation** — 8 tests covering the six documented
  failure modes and two boundary cases (empty payload, existing
  annotations preserved).
- **Annotation format** — 6 tests covering the stability of the
  annotation string contract.

All 24 tests run against mocked calls to the correlation engine — no
live database required.

---

## Lessons Learned

### HTTP Surface Is Not Engineering

The engineering content of this work lives in four places: a pure
enrichment function with graceful degradation, a metric extraction
convention, a stable annotation format contract, and six documented
failure modes with tests. None of these require an HTTP server to
prove.

**Generalization**: when a system produces value at an API boundary,
the boundary is often the *least interesting* engineering decision.
Build the logic first, prove its correctness in tests, and add the
transport layer when it becomes load-bearing.

### Annotation Stability Is Integration Contract

The annotation format is a contract between the system and every
downstream notification channel (chat, paging, email). A change to
the format is a breaking change for every channel that parses it. The
contract is therefore documented and versioned: a prefix
(`correlation_candidate:`) identifies the annotation as originating
here, and the structured portion (`{type=..., score=..., inflection=...}`)
keeps its keys stable across future changes to the underlying fields.

**Generalization**: when a system writes into a shared channel, the
format of the write is a public API. Treat it as such: document the
contract, version it if needed, and never let implementation details
leak into the format.

### Never Block a Notification

A system that enriches alerts should never be able to *suppress* them.
The bare exception handler at the function boundary enforces this:
any unexpected failure returns the alert unchanged. The only way the
enrichment can affect the operator's experience is by adding value —
it cannot subtract.

**Generalization**: for any system layer that sits between a critical
signal and its recipient, the default failure mode should be
pass-through, not interception. "Fail open" is the correct posture
for alerting, not "fail closed."

---

**ADR-019 Status**: Accepted (2026-04-12)
**Supersedes**: None
**Superseded by**: None
