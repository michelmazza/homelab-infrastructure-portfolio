# Phase 18: Predictive Diagnostics — Journey Narrative

**Phase**: 18 — Predictive Diagnostics
**Duration**: 3 weeks, 9 sessions (March 2026)
**Gemini Grade**: A+ (Final)
**Status**: COMPLETE ✅
**Repository**: [homelab-infrastructure-portfolio](https://github.com/michelmazza/homelab-infrastructure-portfolio)

---

## The Challenge: From "What Is" to "What Will Be"

By the end of Phase 17, the homelab RAG system had accumulated impressive analytical
capabilities: hybrid retrieval, query decomposition, multi-hop reasoning, conversational
memory, and conflict detection across documents. What it could not do was look forward.

Given current disk utilization trends, when will a node fill up? Is the memory pressure
on a cluster node stable or escalating? When the system runs heavy analytical queries,
is that overhead showing up in the infrastructure metrics it's supposed to monitor?

Phase 18 answered these questions with a zero-LLM predictive diagnostics engine and
a new Glass Box diagnostic feed — transforming the system from a reactive assistant
into a forward-looking infrastructure observatory.

---

## The Hypothesis

> OLS linear regression over Prometheus metric time-series can reliably detect
> monotonic trends and estimate time-to-threshold, producing actionable predictive
> warnings under a 5ms latency budget, within the zero-LLM constraint.

This hypothesis was validated with Gemini in a Session 0 planning review before
any implementation began. Gemini identified three blind spots to address proactively:
spurious predictions from small sample counts, step-function contamination of
regression windows, and an observer effect correlation capability. All three became
explicit Week 3 deliverables.

---

## Pre-Launch Audit: Measure Twice, Cut Once

Before any implementation sessions began, a 15-minute pre-flight audit of the
`metric_snapshots` database table revealed a critical assumption failure: the table
contained benchmark rows, not infrastructure time-series data. The planning document
had assumed live Prometheus data was already flowing in.

This audit prevented an entire Claude Code session from being consumed on a blocked
task. The amended plan: build the engine with synthetic data in Week 1, wire real
Prometheus scraping in Week 2. "Measure twice, cut once" is the foundational
principle that prevents wasted implementation sessions.

---

## Week 1: Engine Skeleton + Blind Spot Mitigations

**Gemini Grade**: A- (GO for Week 2 with five mandated adjustments)

### Building the Core Engine

The predictive diagnostics engine (`PredictiveDiagnosticsEngine`) was built using
closed-form Ordinary Least Squares (OLS) regression — pure arithmetic, zero ML
libraries, zero LLM calls. The closed-form OLS formula computes a slope (trend
direction and rate) and R² (goodness of fit) from any time-series window in a
single pass. P95 latency on first build: ~0.05ms. Budget: 5ms. Headroom: 100×.

The engine pipeline:

1. **get_snapshots()** — windowed, sorted extraction from the time-series table
2. **compute_ols_slope()** — normalized time-axis regression (prevents floating-point overflow)
3. **compute_cv()** — coefficient of variation on OLS residuals (detrended noise isolation)
4. **check_stale_data()** — multi-reason stale sentinel with named reason codes
5. **detect_step_change()** — deployment jump detection (3× delta threshold)
6. **classify_trend()** — 7 classifications: INCREASING, DECREASING, STABLE, NOISY, STEP_UP, STEP_DOWN, INSUFFICIENT_DATA
7. **estimate_tta()** — time-to-threshold estimation (suppressed when model fit is LOW)
8. **analyze()** — full pipeline orchestrator

**Noise floor validation**: Three synthetic series (disk, memory, CPU) confirmed
the engine correctly classifies stable metrics as STABLE even with realistic
measurement noise, using detrended CV to isolate noise from trend signal.

### Blind Spot Mitigations

Three safety mechanisms added before real data wiring:

**Step-function detection**: Infrastructure metrics don't only trend linearly — they
jump (deployments, restarts, configuration changes). A step change detector flags
any single delta exceeding 3× the average delta magnitude, suppressing TTA
estimation for the affected window.

**Enriched stale sentinel**: Changed from a boolean return to a structured dict with
named reason codes (`null_window`, `all_zero`, `insufficient_points`, `stale_gaps`).
This surfaces in the Glass Box UI as clear, actionable state rather than a silent
suppression.

**Observer effect placeholder**: Added `observer_effect_noted` field to the analysis
output (always `False` in Week 1). The real implementation required live Prometheus
data and query timestamp cross-referencing — both delivered in Week 3.

---

## Week 2: Real Data Pipeline + The CV Reversal

**Gemini Grade**: A+

### Connecting to Live Prometheus

A Prometheus scraper was built targeting the cluster's monitoring endpoint. The
scraper collects 9 metric series (CPU rate, disk available, memory available across
3 infrastructure nodes) at 15-second intervals and persists them to the
`metric_snapshots` table. A platform-native daemon manager runs the scraper
continuously with automatic restart on failure.

### Critical OLS Bug Discovered and Fixed

The initial OLS implementation used raw Unix timestamps as the x-axis (order of
magnitude ~1.7 × 10⁹). This produces floating-point overflow in the OLS denominator,
generating nonsensical regression results. Fix: normalize the x-axis to
seconds-from-first-observation (0, 15, 30, 45...). The slope now represents
units/second in a human-readable scale, and numerical instability is eliminated.

This class of bug — mathematically correct formula, numerically unstable
implementation — is exactly why "validate on real data before declaring done"
is a first-class engineering principle.

### The CV Reversal Finding

After connecting to live infrastructure, the noise floor was re-measured with real
Prometheus data:

| Metric | Synthetic CV (Week 1) | Real CV Range | Stability Factor |
|--------|----------------------|---------------|------------------|
| Disk available | 0.04% | 0.0006–0.0026% | 15–65× more stable |
| Memory available | 0.29% | 0.02–0.07% | 4–14× more stable |
| CPU rate | 17.8% | 0.49–1.83% | 10–36× more stable |

Real 3-node Proxmox infrastructure is dramatically more stable than the synthetic
stress tests assumed. This reversal is a high-confidence validation finding:

- **Engine robustness confirmed**: If the engine handles 65× the noise level of
  real production, it is robust to real-world conditions.
- **Synthetic baselines are conservative by design**: They test worst-case behavior,
  not expected behavior.
- **Calibration signal**: Future alerting thresholds should use real CV ranges as
  the reference point, not synthetic worst-cases.

### R² Confidence Gating

Added R² confidence levels: HIGH (R² ≥ 0.7) / LOW (R² < 0.7). When model fit is
LOW, TTA estimation is suppressed — a trend that doesn't explain its own variance
cannot reliably predict a threshold crossing. On real data, only one metric series
(disk available on one node, R² = 0.99) produced a credible TTA estimate (~278
days). All other series correctly showed LOW confidence. Zero false positives on
stable production infrastructure.

---

## Week 3: Glass Box Tab 7 + Self-Aware Diagnostics

**Gemini Grade**: A+ (Final)

### Glass Box Tab 7

A new tab was added to the RAG chat interface: a live Predictive Diagnostics feed
showing all 9 monitored metric series in real time. Each row displays:

| Column | What It Shows |
|--------|---------------|
| Metric | Series name |
| Trend | INCREASING / DECREASING / STABLE / NOISY / STEP_UP / STEP_DOWN |
| Slope | OLS trend rate |
| CV | Measurement noise level |
| R² | Model fit quality (color-coded: green ≥0.7, yellow 0.4–0.7, red <0.4) |
| Confidence | HIGH / LOW / INSUFFICIENT_DATA |
| TTA | Time-to-threshold range (or "—" if no threshold configured) |
| Observer | 🔴 if the RAG system's own activity is driving this metric |

R² is the key "glass box" signal. It shows the operator exactly how much the
linear model explains the observed data — making the zero-LLM claim transparent
and verifiable at a glance.

### Gemini-Mandated Hardening (All Implemented)

**N≥20 Sample Guard**: Predictions are suppressed when fewer than 20 data points
are available. Legacy metric series with only 1 row correctly show INSUFFICIENT_DATA
in Tab 7 — the guard is visible in production and working as designed.

**Step Honeymoon Fix**: When a step change is detected (deployment, restart), the
OLS window resets to post-step data only. Pre-step slope no longer contaminates
the post-event prediction.

**Production Baseline Synthetics**: 3 new regression anchors using real CV ranges
(not synthetic worst-cases). These verify the engine generates no false critical
warnings on stable, production-realistic data.

**Confidence Interval Funnel**: TTA is displayed as a range (low–high) rather than
a single value. Low confidence widens the range; high confidence narrows it. This
communicates estimation uncertainty without hiding it.

**Retention Policy**: 2,880 rows per metric series (48 hours at 15-second intervals).
Without this cap, continuous scraping would generate ~55,000 rows/day across all
metrics. The cap preserves meaningful OLS history while preventing unbounded database
growth.

### The Observer Effect: Self-Aware Diagnostics

This is the feature Gemini identified as the A+ differentiator.

**The Problem**: In observability systems, the observer perturbs the observed. A
RAG system that answers infrastructure questions — running queries, executing
analytical pipelines, calling APIs — generates its own CPU and memory overhead.
When that overhead shows up as a metric spike, the monitoring system is looking
at its own reflection. This is the "Heisenbug" of infrastructure monitoring: the
act of observation changes the observed system state.

**The Solution**: The observer effect implementation cross-references query activity
timestamps (from the conversation memory database — 458 turns of accumulated
history) against metric snapshots within a ±30-second window. When query-active
periods show statistically higher metric values than quiet periods (above a
one-standard-deviation threshold), the system flags itself as a contributor to
the observed metric change.

**What this looks like in Tab 7**: A 🔴 indicator per metric row. During heavy
analytical query periods, disk and memory rows correctly show 🔴 — the system
acknowledging its own footprint.

**Why this matters for a portfolio**:

> "The system recognizes when its own analytical overhead is responsible for the
> performance changes it is observing." — Gemini Final Validation

This is not a standard monitoring feature. It demonstrates multi-database
correlation, statistical thresholding, and architectural self-awareness. It
answers a question most monitoring systems can't: "Is this a real infrastructure
event, or did I cause it?"

### TTA Shows "—": The Right Decision

Tab 7 shows "—" in the TTA column for all current metric rows. This is by design.

Time-to-arrival estimation only makes operational sense when paired with an explicit
threshold — "disk below 10 GB = alert." Without a defined threshold, a linear
extrapolation is a statistic, not a diagnostic. Displaying "—" is more honest than
displaying a number without operational meaning.

Gemini validated this reasoning explicitly:

> "By requiring a threshold, the system remains a 'Diagnostics Engine' rather
> than a 'Math Toy.' It ensures that every prediction surfaced is actionable."

---

## Results

| Metric | Value |
|--------|-------|
| Predictive engine P95 | ~0.05ms (100× under 5ms budget) |
| R² gating | HIGH ≥0.7, TTA suppressed on LOW |
| Tests added | +90 (718 → 808) |
| Zero regressions | ✅ All 3 weeks |
| Zero-LLM streak | ✅ Phases 16–18 (3 consecutive) |
| Gemini Week 2 | A+ |
| Gemini Final | A+ |
| Real data CV | Disk ~0.002%, Mem ~0.05% (10–65× more stable than synthetic) |
| Prometheus series monitored | 9 (CPU × 3, disk × 3, memory × 3) |

---

## Gemini Final Grade: A+

> "This phase represents a peak in the system's architectural maturity. By moving
> from simple data retrieval to predictive foresight, the system has transitioned
> from a passive assistant to an active infrastructure participant."

On the zero-LLM approach:
> "The Zero-LLM constraint is a performance multiplier, not a limitation."

On the observer effect:
> "This solves a classic problem in observability — the 'Heisenbug' where the
> act of monitoring changes the system state."

Portfolio GO/NO-GO: **GO.**
> "The 'Predictive Diagnostics' narrative is ready. The data is validated,
> the latency is world-class, and the 'Self-Awareness' hook is a unique
> selling point for your infrastructure engineering portfolio."

---

## Key Lessons

**Pre-flight audits prevent wasted sessions.** A 15-minute database check before
launch revealed an empty table — and prevented a session from burning on a blocked
task. The discipline of "validate state before assuming state" is now a first-class
pre-flight gate.

**CV reversal is a validation, not a surprise.** When real infrastructure proved
10–65× more stable than synthetic stress tests, that's not a failure of planning —
it's confirmation that the engine was tested against conservative worst-cases.
Real-world performance will always be better than the stress-test baseline.

**Observer effect data was already there.** The correlation implementation used
conversation history already accumulated across 16 previous phases. No new
infrastructure, no new tables. The best data source is often one already in use
for another purpose.

**Guards should be visible, not silent.** The N≥20 sample guard is not just a
test safety mechanism — it surfaces as INSUFFICIENT_DATA in the live UI, giving
operators clear signal rather than a silently suppressed prediction.

**Silence can be more valuable than a number.** Tab 7 shows "—" for TTA rather
than a number without operational meaning. Honesty about what the system cannot
yet answer is itself a design achievement.

---

## What's Next: Phase 19

**Cross-Modal Diagnostic Correlation (Log-to-Metric)**

Phase 18 answers "The What": disk is filling up. Phase 19 targets "The Why":
disk is filling up *because* a specific service is generating excessive log output.

The build: correlate the metric trends forecasted by the Phase 18 predictive engine
with log events from cluster task and health check logs, using the multi-hop
reasoning infrastructure built in Phase 15. This moves the system from vision to
root cause analysis — from "infrastructure observatory" to "infrastructure
diagnostician."

---

*Phase 18 complete. Gemini Final Grade: A+. Portfolio: GO.*
*The system now tells a story: infrastructure that understands itself and predicts its own future.*
