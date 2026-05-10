# ADR-027: Three-Layer UI Validation — A Testing Standard Encoded From a Real Bug

**Status**: Accepted
**Date**: 2026-04-26
**Phase**: 23 — The Autonomous Bridge
**Grade**: A+ (Gemini CP2 Week 2)

**Counterpart**: `ADR-027-Three-Layer-UI-Validation.md` in this
repository (full architectural detail, real test file references,
verbatim Testing-Guide v5.4 standard, full precedent cross-references).

---

## Context

Phase 23 added a kill-switch UI panel to the Conversational AIOps interface
(see ADR-026). The panel shipped with a bug, was misdiagnosed twice, and
required three iterations to resolve at the architectural root cause rather
than the symptom layer.

What made the bug interesting from a testing perspective: it passed
**function-level tests** (the kill-switch toggle function did the right
thing in isolation) and it passed **AppTest harness tests** (the Streamlit
testing harness rendered the toggle and fired callbacks correctly). It only
surfaced under **manual smoke test**, when an operator interacted with the
chat interface across multiple conversational turns and observed the
safety panel disappearing between turns.

The first fix attempt addressed the wrong layer (widget choice). The second
fix attempt addressed the wrong layer differently (sidebar placement). The
correct fix (Option B — move panel to `main()`) addressed the root cause:
the panel had been rendering only inside an `if prompt:` block, making it
invisible between turns.

This raises a methodological question: if function-level and AppTest
validation can both miss a class of bugs that manual smoke tests catch,
is manual smoke test optional or mandatory? The answer Phase 23 encoded
into a Testing-Guide standard: **mandatory, as a third validation layer
that is never skipped before declaring a UI feature complete.**

---

## Decision

Codify three-layer UI validation as a Testing-Guide standard. All UI
features must pass all three layers before being declared complete:

1. **Layer 1 — Function-level tests**: Does the function do the right
   thing in isolation?
2. **Layer 2 — AppTest integration**: Does the Streamlit harness wire the
   function up correctly? Does state persist? Do callbacks fire?
3. **Layer 3 — Manual smoke test**: Does the operator experience the
   feature correctly in a real session, including across conversational
   turns and across application restarts?

The standard is mandatory. A feature that has passed Layers 1 and 2 but
not Layer 3 is **not complete**. The completion checklist in the
Session 9 workflow explicitly gates on Layer 3 evidence (typically a
session note or screenshot confirming manual validation).

---

## What Each Layer Catches

| Layer | Catches | Misses |
|-------|---------|--------|
| **Layer 1 — Function-level** | Pure logic bugs, return-value contract violations, edge-case input handling | Anything involving Streamlit framework behavior, state persistence, multi-turn rendering, conditional render contexts |
| **Layer 2 — AppTest integration** | Single-turn harness wiring bugs, callback registration, basic state transitions, widget rendering at point of test | Multi-turn rendering bugs, conditional render-context bugs, layout issues, accessibility issues, anything requiring operator perception |
| **Layer 3 — Manual smoke test** | Multi-turn rendering bugs, conditional-render gaps, layout issues, ergonomic issues, "does this actually feel right" issues | Edge-case logic bugs (caught at Layer 1), regression of working behavior (caught at Layer 2 with regression suite) |

Layer 3 is non-redundant precisely because it operates on a different
**evaluator**: a human in a real session, not a test harness. The
kill-switch panel's between-turns invisibility was perceptible to a
human in three seconds and invisible to two layers of automated testing.

---

## Why Not Just Better Automated Tests

A natural objection: "the AppTest harness can be extended to test
multi-turn behavior; manual smoke test is just an automated test we
haven't written yet." This objection has merit and limits.

**Where it has merit**: AppTest *can* be extended to cover multi-turn
sequences, conditional-render contexts, and state transitions. Phase 23
added a multi-step kill-switch lifecycle test to AppTest exactly to
prevent this specific bug from recurring. Automation should always
absorb regression coverage where it can.

**Where it has limits**: Layer 3 is not just "automation we haven't
written yet." It's a different evaluator with a different cost function.

- **Manual smoke test discovers issues automation doesn't know to
  check for.** The between-turns invisibility was not a "bug we needed
  better tests for" — it was a class of behavior nobody had thought to
  test until a human used the feature and noticed it was missing.
- **Manual smoke test evaluates ergonomics, not just correctness.** A
  feature that is correct but confusing fails Layer 3. AppTest cannot
  evaluate confusion.
- **Manual smoke test catches integration with the operator's mental
  model.** The kill-switch is a safety control. A safety control that's
  hidden between turns is correct-but-broken in a way Layer 3 catches
  immediately.

The principle: **automate what regression demands, but always finish
with the human evaluator before declaring complete.**

---

## What "Manual Smoke Test" Means in Practice

A Layer 3 validation pass for a UI feature includes:

1. **Start the application in a fresh state** (not from a cached
   session, not from a partially-initialized state).
2. **Exercise the feature across at least three multi-turn
   interactions** — opening turn, middle turn, closing turn. The
   feature should behave consistently across all three.
3. **Observe what happens between turns** — does the feature remain
   present? Does state persist correctly? Does the feature degrade
   gracefully on errors?
4. **Exercise the feature across an application restart** — do
   persistent states survive? Do session-scoped states reset
   appropriately?
5. **Capture evidence** — a session note, a screenshot, or both. The
   evidence is committed to the session folder as part of the
   Session 9 completion checklist.

The cost: ~3-5 minutes per feature for a typical Layer 3 pass. The
benefit: catches the entire class of bugs that automation systematically
misses.

---

## Integration with Session Workflow

The Testing-Guide.md v5.4 standard wires Layer 3 into the Session 9
completion gate:

- A UI feature without Layer 3 evidence is **not eligible for**
  inclusion in the session completion summary.
- The session-N-summary.txt must reference the manual smoke test
  evidence path (session note, screenshot, or both).
- The session cannot be declared complete until the evidence is
  captured and committed.

This is not "documentation overhead." It's the same pattern as the
Session Summary Gate (CHAT-SESSION-WORKFLOW v1.6) — a process control
that prevents an entire class of post-completion bugs by gating
declaration on evidence rather than intent.

---

## Phase 17 Precedent

Phase 17 had previously discovered a related lesson: integration tests
that pass at the unit level can hide whether a feature is wired into
the application. Phase 17 surfaced three Glass Box tabs that were
implemented as render functions but never called from the application
loop. Tests passed because the render functions did the right thing
when called; the application never called them.

Phase 17 added the **UI Smoke Test Gate** to the workflow. Phase 23
extended this from "smoke test before completion report" to "three
layers of validation, all required, smoke test as Layer 3." The
phases agree on the principle: integration cannot be declared complete
on the basis of unit tests alone.

---

## Rejected Alternatives

### Manual smoke test as optional

**Rejected**: This is the de facto state in many UI projects. It's
also why UI bugs ship at high frequency. Phase 23 ships demonstrate
that two layers of automated validation can both pass on a feature
that is broken in a way an operator perceives in three seconds.
Optional manual validation degrades to never-performed manual
validation under deadline pressure.

### Two-layer standard (function + AppTest)

**Rejected**: This is what Phase 23 had before the kill-switch bug.
The bug shipped under that standard. The lesson encoded in this ADR
is that two layers are insufficient when the omitted layer is the
one that catches a distinct class of bugs.

### Manual smoke test only on "user-facing" features

**Rejected**: The distinction between "user-facing" and "internal" is
fuzzy in a system where the chat interface is the operator's primary
interaction. Better to apply the standard to all UI features and
accept the small cost than to litigate the boundary case-by-case.

### Replace AppTest with manual testing

**Rejected**: AppTest catches regressions automation should catch
(callback wiring, state persistence at single-turn level, basic
rendering correctness). Removing it shifts that burden to manual
testing, which is more expensive and more error-prone for regression
coverage. The right answer is "all three layers," not "any one layer
plus the others optional."

### Smoke test by recording demos

**Rejected briefly**: A recorded demo could provide Layer 3 evidence
without requiring repeated manual passes. Concluded this would degrade
to "the operator records a demo once and never re-validates," which
is exactly the failure mode the standard prevents. Manual smoke test
must be performed on each completion, not replayed from a recording.

---

## Results

| Metric | Result |
|--------|--------|
| Validation layers required | 3 (function, AppTest, manual smoke) |
| Layers the kill-switch bug passed | 2 (function, AppTest) |
| Layers the kill-switch bug failed | 1 (manual smoke) |
| Design iterations needed when Layer 3 was skipped | 3 |
| Time to identify root cause once Layer 3 was performed | Single observation |
| Time cost per feature for Layer 3 pass | ~3-5 minutes |
| Codified into | Testing-Guide.md v5.4 |
| Workflow integration | Session 9 completion gate (CHAT-SESSION-WORKFLOW) |
| External grade | A+ (Gemini CP2 Week 2) |

---

## Consequences

**Positive**:

- An entire class of UI bugs (multi-turn rendering gaps, conditional-
  render context bugs, ergonomic issues) is now systematically caught
  before completion.
- The standard is process-encoded, not memory-dependent. Future
  contributors inherit the discipline through the Testing-Guide,
  not through tribal knowledge.
- The standard generalizes beyond Phase 23 — it's a project-wide
  Testing-Guide standard applicable to all future UI work.
- Layer 3 evidence (session notes, screenshots) becomes part of the
  permanent session record, providing reviewable validation evidence
  for future audits.
- The standard creates a forcing function that aligns implementation
  effort with operator-perceived correctness, not just unit-test
  correctness.

**Accepted tradeoffs**:

- Manual smoke testing has a per-feature cost (~3-5 minutes). This is
  judged worth it for the bug class it catches.
- Manual smoke testing is harder to automate against, by definition.
  Layer 2 (AppTest) absorbs as much regression coverage as it can,
  but Layer 3 remains a human evaluator step.
- The standard increases the floor cost of UI features. Trivial UI
  changes still require Layer 3 evidence. This friction is intentional —
  the alternative is selectively applying the standard and reintroducing
  the failure mode.

---

## External Validation

> "Codifying the lesson as a Testing-Guide standard is the mark of
> mature engineering. The bug shipped twice; the methodology innovation
> that emerged ensures it cannot ship a third time. Three-Layer UI
> Validation is a portfolio-grade pattern that generalizes well beyond
> this specific incident."
>
> — Gemini CP2 (Week 2)

The standard was characterized as "lesson encoded as a process" — a
design pattern that converts a single-incident learning into a
permanent process control. This is the same pattern Phase 17 used
for the UI Smoke Test Gate, Phase 21 used for the Sanitization
Completeness Checklist, and the multi-phase workflow has used for
the Session Summary Gate.

---

## Related Decisions

- **ADR-026** (Phase 23): Kill-Switch with Confirmation Gate — the
  feature whose bug history motivated this ADR
- **CHAT-SESSION-WORKFLOW v1.6**: Session Summary Gate — sibling
  process control that gates session completion on evidence
- **CHAT-SESSION-WORKFLOW v1.5**: Pre-Execution Gate — task list
  presentation before implementation begins
- **Phase 17 UI Smoke Test Gate**: The earlier precedent that
  established smoke testing as a completion gate for UI features

---

## Forward Carry

The standard applies to all future UI features. Phase 24's planned
Tab 8 stale-UI banner, the FastAPI webhook receiver UI surfaces (if
any), and any future Glass Box tab additions are expected to provide
Layer 3 evidence as part of their completion gate.

The pattern also generalizes beyond UI. The principle — "automation
catches what automation knows to look for; humans catch what nobody
thought to look for" — applies to integration testing, deployment
validation, and operational readiness reviews. The specific Three-
Layer formulation is UI-scoped, but the underlying discipline
(automate where you can, finish with a human evaluator) is broader.

---

**ADR**: 027
**Phase**: 23 — The Autonomous Bridge
**Status**: Accepted
**Grade**: A+ (External Validation)
**Created**: 2026-04-26
**Portfolio Version**: This document (sanitized for public repository)
**Private Counterpart**: Phase 23 Completion Report §5.4, §7.1 +
Testing-Guide.md v5.4 Three-Layer UI Validation section
