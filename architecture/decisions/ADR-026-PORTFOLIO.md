# ADR-026: Kill-Switch with Confirmation Gate — Functional Safety Over Widget Elegance

**Status**: Accepted
**Date**: 2026-04-26
**Phase**: 23 — The Autonomous Bridge
**Grade**: A+ (Gemini CP2 Week 2)

**Counterpart**: `ADR-026-Kill-Switch-Confirmation-Gate.md` in this
repository (full architectural detail, real module paths, real line
numbers, code snippets, three-iteration narrative).

---

## Context

Phase 23 introduced live execution of remediation actions through the 5-gate
pipeline (ADR-025). A live execution path needs a global emergency brake —
an operator-controlled mechanism that can halt all live actions across all
patterns, evaluated before alert routing and persisted across application
restarts.

The kill-switch had two surfaces:

1. **Command-line interface**: a simple `--kill-switch on|off|status` flag
   for operators working in a terminal session. Scriptable, automation-
   compatible, suitable for inclusion in operational runbooks.
2. **UI panel**: a dedicated Glass Box tab section in the Conversational
   AIOps interface, for operators working in the chat interface. Always
   visible. State-persistent across conversational turns.

The CLI was straightforward. The UI was not. The kill-switch UI shipped with
a bug, was misdiagnosed twice, and ultimately produced a methodology
innovation (Three-Layer UI Validation, ADR-027) and an architectural lesson
about confirmation gates.

This ADR documents the kill-switch design decisions, the asymmetric
confirmation pattern, and the Option B architectural fix that closed the bug
at the root cause rather than the symptom layer.

---

## Decision

The kill-switch uses **asymmetric confirmation**:

- **Enable** (the safe direction): single click, no confirmation. Disabling
  Level 2 actions immediately is always safe and should be frictionless.
- **Disable** (the unsafe direction): click followed by an explicit
  confirmation modal. Re-enabling Level 2 actions admits autonomous
  infrastructure mutations and warrants a deliberate confirmation step.

The UI panel is rendered at the top level of the chat interface's `main()`
function — **not inside any conditional render block**. This makes the
safety control always visible, regardless of conversational state.

State is persisted to a JSON state file and consulted by the alert-routing
layer before forwarding to the live execution pipeline. CLI and UI share
the same state file: a CLI toggle is reflected in the UI on next render,
and vice versa.

---

## The Three Design Iterations

The kill-switch UI required three design iterations before landing. Each
iteration revealed a different layer of the problem.

### Iteration 1 — `st.toggle` widget

Initial implementation used a single `st.toggle` widget with `kill_switch_active`
as the toggle state. Function-level tests passed. AppTest harness tests passed
(harness rendered the toggle, callbacks fired). Manual smoke test surfaced the
bug: the toggle state appeared to carry forward across reruns in a way that
bypassed the confirmation gate flow.

The misdiagnosis: "this is a `st.toggle` widget behavior issue."

### Iteration 2 — Sidebar placement

Second attempt moved the toggle from the main chat area to the sidebar,
hypothesizing that placement was producing the carry-forward behavior.
Function-level tests passed. AppTest passed. Manual smoke test still failed.
The sidebar placement did not address the root cause.

The misdiagnosis: "this is a placement issue, but maybe the wrong placement."

### Iteration 3 — Option B (panel to `main()`, button pair, asymmetric confirmation)

The actual root cause: the panel had been placed inside the chat interface's
`if prompt:` block. That block only renders on turns following a user message.
**Between turns, the safety control disappeared entirely.** Function-level
tests cannot catch that. AppTest cannot catch that without a multi-turn
interaction sequence. Only a manual smoke test surfaced it.

The Option B fix had two parts:

1. **Move panel to `main()`**: Render unconditionally on every Streamlit run,
   not just on prompt turns. This addressed the visibility root cause.
2. **Replace `st.toggle` with explicit Enable/Disable button pair plus
   asymmetric confirmation**: The button pair gave precise click-to-action
   mapping, removed the carry-forward state behavior of `st.toggle`, and
   admitted the confirmation gate naturally.

The bug closed. The methodology lesson encoded as ADR-027.

---

## Why Asymmetric Confirmation

A symmetric confirmation pattern would have required confirmation on both
Enable and Disable. This is a common reflex but the wrong design choice.

**The asymmetry tracks the actual safety risk**:

| Direction | Effect | Safety Risk | Confirmation? |
|-----------|--------|-------------|---------------|
| **Enable kill-switch** | Halts all live actions | Low — system becomes more conservative | None — single click |
| **Disable kill-switch** | Re-admits live actions | Higher — system mutates infrastructure again | Confirmation modal |

Frictionless Enable means an operator who notices something wrong can halt
the system instantly. A confirmation prompt at this moment would itself be
an operational hazard — operators racing the system shouldn't have to read
modal text. Frictionless gating in the dangerous direction means the
operator must take an explicit, deliberate step to admit autonomous
mutations again.

This asymmetry recurs in production safety design: emergency stops on
industrial equipment are large and unconfirmed; restart procedures involve
checklists. The kill-switch follows the same principle.

---

## Why Button Pair Over Toggle

`st.toggle` is the visually elegant choice for a binary state control. It
was the wrong choice for this use case. Three reasons:

**State carry-forward semantics**: Streamlit's toggle widget retains state
across reruns in a way that does not compose well with explicit confirmation
flows. A button click is a discrete event; a toggle state is a continuous
property. Confirmation flows want discrete events.

**Click-to-action mapping is implicit with toggles**: A toggle implies "this
state is now X." A button pair makes the action explicit: "you clicked
Enable" or "you clicked Disable." Under AppTest, this distinction matters —
button clicks are easier to reason about than state transitions.

**Asymmetric confirmation is harder to express on a toggle**: Building
"Enable is free, Disable is gated" on top of a single toggle widget requires
intercepting the state change and selectively reverting it on the unsafe
direction. The button pair expresses the same logic as two distinct
event handlers, which is structurally cleaner and easier to test.

The general principle: **functional safety over widget elegance**. A control
that mediates infrastructure mutations is not the place to optimize for
visual minimalism.

---

## The Confirmation Gate Implementation

The Disable path:

1. Operator clicks **Disable Kill-Switch**.
2. UI renders confirmation modal: "Confirm: disable kill-switch and
   re-enable Level 2 actions?"
3. Modal presents two buttons: **Confirm Disable** and **Cancel**.
4. **Confirm Disable**: state file updated to `disabled`, modal closes,
   panel re-renders showing kill-switch disabled.
5. **Cancel**: modal closes, no state change, kill-switch remains in its
   prior state.

The Enable path skips all of this — single click writes the state file
and re-renders the panel.

**Persistence guarantee**: The state file is written before the UI
re-renders. If the application crashes between the click and the next
render, the state is already persisted. The next startup reads the file
and shows the correct kill-switch state.

---

## Always-Visible Placement (Option B Detail)

The panel is rendered at the top of `main()`, before any conditional
content blocks. This guarantees:

- The kill-switch is visible on the first turn of a session, before any
  user message has been sent.
- The kill-switch is visible between turns, while the operator reads
  output and considers the next message.
- The kill-switch is visible during turns, while the system is processing
  a message.
- The kill-switch is visible after the session ends, until the operator
  navigates away.

A safety control that is sometimes visible is, for the moments it is not
visible, equivalent to a safety control that does not exist. The Option B
fix removed all moments-not-visible from the kill-switch panel's lifecycle.

---

## Rejected Alternatives

### Skip the kill-switch, rely only on pattern-level flags

**Rejected**: Pattern-level `live_enabled` flags require editing YAML
files, committing changes, and reloading the application. A global
operator override that takes effect immediately, without code changes,
is a different control with a different scope. Both are needed.

### Confirmation on both directions (symmetric gate)

**Rejected**: Confirmation friction in the safe direction is itself an
operational hazard. The asymmetry tracks the actual risk asymmetry.

### `st.toggle` with manual carry-forward state management

**Rejected**: Building reliable confirmation flows on top of `st.toggle`'s
state semantics requires intercepting state changes and conditionally
reverting them. This is harder to test, harder to reason about, and
harder to verify under AppTest than the button-pair design. The button
pair expresses the logic naturally.

### Sidebar placement (Iteration 2 attempt)

**Rejected**: Did not address the root cause. The bug was conditional
rendering inside `if prompt:`, not the chosen widget container. Sidebar
placement would have moved the panel to a different location while
preserving the visibility gap.

### Confirmation via separate page

**Rejected**: A safety control that requires navigation is, in operational
moments, equivalent to a safety control that doesn't exist. The
confirmation modal stays in-place; the operator does not navigate away
from the chat interface to use it.

### Indefinite confirmation timeout

**Rejected**: Considered briefly. Concluded that an explicit operator
**Cancel** is cleaner than a silent timeout that admits ambiguity ("did
the system honor my intent or did it time out?"). The modal stays open
until the operator dismisses it.

---

## Results

| Metric | Result |
|--------|--------|
| Surfaces | 2 (CLI + UI) |
| Persistence | JSON state file, shared across surfaces |
| Confirmation pattern | Asymmetric (Enable: free, Disable: gated) |
| UI placement | Top of `main()`, always visible |
| Widget type | Button pair (not toggle) |
| Design iterations to land | 3 |
| Validation layers required to catch the placement bug | 3 (function, AppTest, manual) |
| Tests added (kill-switch lifecycle AppTest) | 1 multi-step test |
| External grade | A+ (Gemini CP2 Week 2) |

---

## Consequences

**Positive**:

- Operator can halt all live actions instantly with a single click.
- Operator cannot accidentally re-enable live actions — explicit confirmation
  required.
- Safety control is visible at every moment of the operator's session.
- CLI and UI share state; toggling either surface is reflected in the
  other on next observation.
- State persists across application restarts, which means the kill-switch
  cannot be silently reset by a deploy or a crash.
- Pattern serves as the canonical example of "functional safety over
  widget elegance" for future UI safety controls.

**Accepted tradeoffs**:

- Always-visible placement consumes screen real estate that could be used
  for other UI panel content. This is judged worth it for a safety control.
- Asymmetric confirmation is a non-obvious pattern; new contributors
  encountering this code may instinctively want to symmetrize it. The
  asymmetry is intentional and documented.
- Modal-based confirmation interrupts the operator's flow. This friction
  is the point — it's friction in the unsafe direction only.

---

## External Validation

> "Three independent safety controls operating in series — pattern-level
> flag, global kill-switch, stable-duration rollback — provide defense in
> depth without coordination overhead. The kill-switch with confirmation
> gate exemplifies the principle of asymmetric friction: free to make
> safer, gated to make less safe."
>
> — Gemini CP2 (Week 2)

Gemini CP2 also flagged the misdiagnosis-to-resolution arc as a portfolio-
grade narrative: a UI bug that escaped two layers of validation, was
misdiagnosed twice, and ultimately produced a methodology innovation
(Three-Layer UI Validation). The lesson is "encoded as a process," which
Gemini characterized as the mark of mature engineering.

---

## Related Decisions

- **ADR-025** (Phase 23): 5-Gate Live Execution Pipeline — the kill-switch
  is one of three independent safety controls operating against this
  pipeline
- **ADR-027** (Phase 23): Three-Layer UI Validation — the testing
  methodology that emerged from this ADR's bug history
- **Phase 22 Kill-Switch (CLI-only)**: The Level 1 dry-run kill-switch
  that this ADR extends with UI surface and confirmation gate

---

## Forward Carry

The asymmetric confirmation pattern generalizes beyond the kill-switch.
Any UI control that admits an operationally hazardous state transition
should be considered for asymmetric friction: free to make safer, gated
to make less safe. Future UI panel controls (pattern enable/disable, rate
limit overrides) should default to this pattern unless there's an
explicit reason to symmetrize.

The Option B placement principle — render safety controls at the top
level of `main()`, never inside conditional blocks — applies to all
future safety controls regardless of widget choice.

---

**ADR**: 026
**Phase**: 23 — The Autonomous Bridge
**Status**: Accepted
**Grade**: A+ (External Validation)
**Created**: 2026-04-26
**Portfolio Version**: This document (sanitized for public repository)
**Private Counterpart**: Phase 23 Completion Report §5.2, §5.3, §6.1, §6.3
