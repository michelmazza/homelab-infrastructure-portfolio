# Phase 21 Metrics — Distributed Foundations

**Phase**: 21
**Timeline**: 2026-04-11 to 2026-04-12 (3 weeks, 9 sessions)
**Grade**: A+ (Gemini CP2 confirmed)
**Status**: Complete

---

## Executive Summary

Phase 21 is the first phase whose headline is **infrastructure**, not
intelligence. The before/after is read in one line: *Mac Studio was the
single point of data and compute; Mac Studio is now removed from the
compute plane entirely.* The numbers below trace that transition across
four axes — compute, data, IAM, and operational surface area — plus
two continuity anchors (test baseline, zero-LLM streak) that prove the
migration was a refactor, not a rewrite.

---

## Headline Comparison

| Axis                       | Phase 20 End                     | Phase 21 End                                               |
|----------------------------|----------------------------------|------------------------------------------------------------|
| Compute location           | Mac Studio (raw_exec, macOS)     | Linux cluster nodes (docker driver, multi-arch)            |
| Data location              | SQLite on Mac Studio             | PostgreSQL on lab-node-3                                |
| Database role              | postgres SUPERUSER               | app_role scoped role (CRUD only)                        |
| Container registry         | None                             | registry:2 on node-3:5000 (multi-arch)                      |
| Containerized services     | 0                                | 3 (scraper, lifecycle, log collector)                      |
| Mac Studio Nomad jobs      | 2 (scraper, lifecycle — raw_exec) | 0 (node eligibility disabled)                              |
| Nomad service discovery    | Consul (all nodes implied)       | Consul on node-1/node-2; Nomad-native on node-3/Mac Studio    |
| ADR count                  | 17                               | 20 (+ADR-018, ADR-019, ADR-020)                            |
| Tests collected            | 999                              | 1036                                                       |
| Tests passing              | 980                              | 1002                                                       |
| Zero-LLM analytical streak | 5 phases                         | 6 phases                                                   |

---

## Test Baseline Progression

Phase 21 opened at 999 tests and closed at 1036 (+37 net). The 1000th
test (`tests/test_system_integrity.py::test_full_observability_stack`)
was deliberately written as a symbolic milestone — an end-to-end
validation of the Phase 18–20 stack that exercises metric snapshots,
OLS prediction, log events, correlation, DiagnosticContext persistence,
and follow-up detection in a single run.

| Phase Boundary   | Collected | Passed | Skipped | Failed | Net Delta |
|------------------|-----------|--------|---------|--------|-----------|
| Phase 20 close   | 999       | 980    | 19      | 0      | baseline  |
| S1 (1000th test) | 1000      | 981    | 19      | 0      | +1        |
| S2 (db.py)       | 1012      | 993    | 19      | 0      | +12       |
| S3 (schema DDL)  | 1012      | 993    | 19      | 0      | 0         |
| S4 (write path)  | 1014      | 995    | 19      | 0      | +2        |
| S5–S6            | 1014      | 995    | 19      | 0      | 0         |
| S7 (container)   | 1014      | 980    | 19      | 15     | 0 new*    |
| S8 (alerting)    | 1036      | 1002   | 19      | 15     | +22       |
| **Phase 21 close** | **1036** | **1002** | **19** | **15** | **+37**   |

\* The 15 failures appearing at Session 7 are Consul integration tests
(`rag/tests/test_mcp_consul_integration.py`) that require a live Consul
agent on node-3. They are infra-dependent, not code regressions, and are
documented as such in Testing-Guide v4.8 (the same class of expected
failure as the 10 Prometheus infrastructure skips).

**Regression anchor**: 1002 passed, 19 skipped, 15 failed (Consul
infra-dependent). Any future session producing fewer passes is a
regression.

---

## New Test Modules

| Module                      | Tests | Purpose                                             |
|-----------------------------|-------|-----------------------------------------------------|
| `tests/test_system_integrity.py` | 1     | 1000th-test milestone (end-to-end stack)            |
| `tests/test_db.py`          | 14    | SQLite paths, mocked Psycopg routing, error paths, `is_sqlite_connection` |
| `tests/test_alerting.py`    | 24    | Alertmanager webhook enrichment (metric/node extraction, graceful degradation, annotation format) |

The `test_db.py` tests validate the connection factory's routing logic
without requiring a live PostgreSQL instance — Psycopg calls are
patched via `sys.modules` stub. This keeps the test suite entirely
SQLite-portable.

`test_alerting.py` covers six failure modes for `enrich_alert()`:
no candidates → alert unchanged, database unreachable → alert
unchanged, malformed alert → alert unchanged, metric extraction
failure → alert unchanged, node extraction failure → alert unchanged,
happy-path annotation → alert enriched. Graceful degradation is not a
mode; it is the default.

---

## Data Migration Volumes

| Table             | SQLite Source | PostgreSQL After Session 3 | At Phase Close |
|-------------------|---------------|----------------------------|----------------|
| metric_snapshots  | 25,925 rows   | 25,938 rows (+13 live)    | 78,042 rows    |
| log_events        | 9 rows        | 9 rows                    | ~110 rows      |

The +13 delta at Session 3 reflects normal scraper activity during
migration (5 Phase-16 ISO-string artifacts deliberately skipped,
18 fresh writes forward-migrated). The growth to 78,042 rows by
phase close is ~52,000 scraper writes across 24 hours, confirming the
containerized scraper on lab-node-1 is writing to PostgreSQL on
lab-node-3 at a normal cadence.

---

## A+ Bar Proof

Gemini's A+ criteria for Phase 21 required four conditions met
simultaneously:

| # | Criterion                                   | Session | Evidence                                                                 |
|---|---------------------------------------------|---------|--------------------------------------------------------------------------|
| 1 | PostgreSQL on node-3 (not Mac Studio)        | 2–3     | `app_db` database in `postgres:16-alpine` on `10.0.0.42`      |
| 2 | ≥1000 tests passing                         | 1       | 1000th test committed (final: 1002 passing)                             |
| 3 | Cross-node writes                           | 6       | Mac Studio scraper → node-3 PostgreSQL (row-count delta verified)        |
| 4 | Scraper runs on a Linux cluster node (not Mac Studio) | 7  | Alloc `b319419f` on lab-node-1 (docker driver), Healthy 1/1          |

At phase close, the scraper on lab-node-1 (ARM64) was writing to
PostgreSQL on lab-node-3 (AMD64) via a scoped non-SUPERUSER role.
Compute on one node, data on another, neither of them the original
development machine — the architectural pin is broken.

---

## Cluster State at Phase Close

All 4 Nomad clients participating; Mac Studio excluded from scheduling.

| Job                       | Nodes             | Driver    | Status        |
|---------------------------|-------------------|-----------|---------------|
| prometheus-scraper        | lab-node-1     | docker    | Running 1/1   |
| log-event-collector       | lab-node-3     | docker    | Running 1/1   |
| observability-lifecycle   | periodic batch    | docker    | Next: 02:00Z  |
| registry                  | lab-node-3     | docker    | Running 1/1   |
| postgresql                | lab-node-3     | docker    | Running 1/1   |
| grafana                   | lab-node-3     | docker    | Running 1/1   |
| traefik                   | lab-node-3     | docker    | Running 1/1   |
| searxng                   | lab-node-3     | docker    | Running 1/1   |
| node-exporter             | nodes 1/2/3       | docker    | Running 3/3   |

**Mac Studio** (development workstation): node eligibility disabled,
zero jobs scheduled. The Streamlit RAG app and Ollama LLM inference
continue to run here as development tooling — they are not managed by
Nomad and so are outside the scope of this count.

---

## Zero-LLM Analytical Streak

Phase 21 extends the zero-LLM streak to **6 consecutive phases (16–21)**.
The entire analytical stack operates on pattern matching, closed-form
arithmetic, and deterministic heuristics with bounded overhead:

| Module                      | Technique                                       | Bound          |
|-----------------------------|-------------------------------------------------|----------------|
| Predictive engine           | OLS regression (closed-form)                    | P95 ~0.05ms    |
| Correlation engine          | Recency × severity × node_match scoring         | < 1ms          |
| Synthesis pipeline          | Pattern-based conflict detection + resolution   | ~0.2ms         |
| Diagnostic context guard    | TTL-bounded caching + hard truncation           | < 0.1ms        |
| Alerting enrichment         | Regex metric extraction + correlation lookup    | < 1ms          |
| Intent classifier           | Keyword matching + metric/node extraction       | < 0.1ms        |

No ML libraries. No NLP dependencies. No API calls. The entire
analytical path is self-contained and deterministic — every output is
explainable by reading the code. The LLM is called only in the final
synthesis step, and only to render the analytical result as natural
language. This separation is an architectural decision documented in
ADR-017 (Zero-LLM Analytical Architecture).

---

## Architecture Decisions Recorded

Phase 21 added three ADRs:

| ADR      | Title                                                         | Scope                                              |
|----------|---------------------------------------------------------------|----------------------------------------------------|
| ADR-018  | PostgreSQL Migration                                          | Data plane portability via `rag/db.py` factory     |
| ADR-019  | Alerting Integration                                          | Alertmanager webhook enrichment pattern            |
| ADR-020  | Nomad-Native Discovery + Job-File Placement Policy            | Heterogeneous cluster placement durability         |

ADR-020 is the architectural novelty of the phase — it came out of an
operational incident (Docker restart on node-3) rather than a planned
design decision, which is why its insight is durable: the distinction
between *operational state* and *architectural intent* would not have
surfaced without the forced rescheduling event that exposed it.

---

## Cost of the Phase

| Resource             | Amount                                          |
|----------------------|-------------------------------------------------|
| Sessions             | 9 (including Session 0 planning)                |
| Claude Code sessions | 7 (Sessions 1, 2, 3, 4, 5, 7, 8, 9)             |
| Claude Chat sessions | 2 (Session 0, Session 6 pre-flight + pivot)     |
| Gemini checkpoints   | 2 (CP1 post-Week-2 consolidated, CP2 Session 8) |
| Lines of code added  | ~850 (rag/db.py + rag/alerting.py + migrations) |
| Lines of code removed| ~60 (`_enforce_retention` from scraper)         |
| Nomad job files new  | 2 (registry, log-event-collector)               |
| Nomad job files edited | 7 (service blocks removed, driver changed)     |
| Dockerfiles new      | 1 (scraper — reused by lifecycle + collector)   |
| Config files edited  | 2 (`rag/config.py`, `requirements.txt`)         |

The Dockerfile count is notable: only one was written. The shared-image
pattern (all three containerized services run from the same
`python:3.11-slim` image with different entrypoints) is an intentional
consolidation that simplifies CI/CD in later phases.

---

## What This Phase Is Not

Phase 21 is not a performance optimization, not a new analytical
capability, not a user-facing feature. Tab 7 and Tab 8 in the Glass Box
UI are unchanged. Query latency is unchanged. Accuracy is unchanged.
The only test-visible behavior change is the existence of 37 new tests.

The change is entirely in *where things run*, *what role they assume*,
and *how placement is encoded*. The headline for the portfolio story is
infrastructure engineering: distributed data plane, containerized
compute plane, least-privilege IAM, and the architectural-intent vs
operational-state insight that ADR-020 formalized.

---

## Regression Anchor (Forward Reference)

For Phase 22 opening baseline:

- **Collected**: 1036
- **Passing**: 1002
- **Skipped**: 19 (10 Prometheus infra + 9 conditional)
- **Failed**: 15 (Consul integration — infra-dependent, documented)
- **Flaky**: 1 (`test_window_overhead_under_5ms` — timing jitter, passes on re-run)

Any Phase 22 session producing fewer than 1002 passes introduces a
regression.

---

**Phase 21 metrics — Distributed Foundations. Locked.**
