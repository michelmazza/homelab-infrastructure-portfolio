# Phase 21 Journey — Distributed Foundations

**Phase**: 21
**Duration**: 3 weeks, 9 sessions
**Grade**: A+ (external validation)
**Methodology**: Benchmark → Experiment → Validate → Pivot

---

## The Shape Problem

Every phase before this one was an intelligence phase. Earlier work
made retrieval smarter, planning smarter, memory smarter,
observability smarter. By the end of the previous phase, the system
could detect metric inflections, correlate them with infrastructure
events, persist diagnostic context across conversation turns, and
render the entire reasoning trace in an eight-tab "Glass Box" UI.

And all of it ran on a single development workstation.

Not *some* of it. *All of it.* Every observability job in the
cluster carried a placement constraint pinning it to that one host,
because the database file lived on the workstation's local
filesystem and writing to it required local access. The job file for
the metric scraper said, in effect, "this job runs on the
workstation, forever." Same for the log event collector. Same for
the retention manager.

External feedback at the previous phase's portfolio checkpoint had
flagged this as "the single largest architectural debt item." It
was not a performance problem. It was not an accuracy problem. The
system worked fine. It was a *shape* problem — the system was shaped
like a single-node project that happened to call itself distributed.

Phase 21 was the phase to fix the shape.

---

## The Hypothesis

Session 0 planning framed the work as two testable hypotheses.

The first: **migrating from a local SQLite file on the workstation to
PostgreSQL on a dedicated cluster node, via a backend-agnostic
connection factory, can be accomplished in three weeks without
regressing the current test baseline.** If true, every observability
module would work against either backend with the same code.

The second: **surfacing the correlation engine's Top Cause output in
monitoring alerts via a webhook enrichment pattern can be implemented
in two sessions.** If true, the system would transition from a
passive observer (answers live in the UI) to an active participant
(answers reach the operator at the point of need).

Three supporting conditions framed both hypotheses:

- **Zero-LLM analytical streak must hold**: the analytical path has
  been LLM-free for five consecutive phases. Phase 21 extends to 6.
- **The workstation must end up not running any cluster jobs**: not
  "fewer jobs", not "some jobs migrated" — zero.
- **The thousandth test must be a meaningful end-to-end validation**,
  not a trivial no-op written to hit a round number.

External validation at Session 0 defined the A+ bar:
**"Computational Portability — scraper running on a Linux cluster
node, database on a different cluster node, all ≥1000 tests passing
while the scraper writes across the network."** Four components.
All four had to be met simultaneously.

---

## Week 1 — Foundation

**Thesis**: Build the data plane without disrupting the running system.

Week 1 was non-disruptive by design. Every existing test and module
continued to read and write the embedded database. PostgreSQL would
become load-bearing only when Week 2 flipped individual modules onto
the new backend.

Session 1 delivered the thousandth test as a single end-to-end
integration test exercising the entire analytical stack — metric
snapshots, regression-based prediction, log events, correlation,
persistent diagnostic context, follow-up detection. If any part of
the pipeline breaks, this test fails.

Session 2 ran into its first unplanned reality. The starter prompt
assumed a greenfield PostgreSQL deploy. Pre-flight inspection found
a pre-existing instance that had been running on the target node for
eleven days, serving three other homelab services. A literal reading
of the starter prompt would have destroyed all three.

The pivot took ten minutes. Instead of provisioning a new instance,
the project would provision a fourth logical database inside the
running one, use the existing administrative role for Week 1
velocity, and plan a least-privilege rotation for Week 3. This is
the kind of decision that does not show up in a success metric. The
actual engineering decision was *not* to follow the plan as written,
because following it would have caused cross-service data loss. The
preflight check was the load-bearing step, and it took one command.

Session 2's technical output was a 137-line connection factory
module that routes database access based on URL scheme. The factory
exposes a `get_connection()` function and an `is_sqlite_connection()`
predicate; an internal wrapper translates PostgreSQL placeholders to
SQLite on every execute. Consumer modules write a single dialect;
translation happens in one place.

Session 3 delivered the schema and data migration. BRIN indexes on
timestamp columns (an external recommendation for time-series
workloads). An idempotent migration driver with an isolation guard
that asserts the target database name before any DML — cheap
insurance against the administrative-role footgun.

### The Checkpoint Skipped

Standard methodology calls for an external validation checkpoint at
the end of Week 1. I skipped it. The rationale at the time: Week 1
was non-disruptive, the baseline was unchanged, Week 2's evidence
would strengthen a consolidated submission.

This was a mistake. Not a fatal one, but a costly one. I did not
know it yet.

---

## Week 2 — Module Cutover

**Thesis**: Port every observability code path to backend-agnostic
SQL; prove cross-node writes; remove the placement pin.

Sessions 4 and 5 systematically migrated five observability modules
onto the connection factory. The pattern was mechanical for each
module: replace direct database connection calls with the factory,
change placeholder style, adjust DDL types, centralize
connection-mode settings in the wrapper.

The interesting architectural decision was in Session 5's retention
query port. SQLite uses one timestamp-arithmetic syntax; PostgreSQL
uses a different one. Both required dialect branching unless we
could find a way to avoid it.

The resolution turned out to be the single most impactful line of
the migration:

```python
cutoff = int(time.time()) - seconds
```

Compute the cutoff in Python. Pass it as a bound parameter. Both
backends now receive the same SQL with the same parameter. The
dialect surface area for temporal queries drops to zero. This one
decision eliminated the largest class of branching the codebase
would have otherwise needed.

One dialect divergence resisted the Python-side treatment: JSON
extraction. SQLite and PostgreSQL use different syntax for it, and
the value being extracted is computed on the database side. For
this case, the narrow dialect-gate pattern won: fork on the
connection type predicate, keep both branches explicit.

### Session 6 — The Pivot

Session 6 was supposed to be the pin-removal session. Remove the
placement constraint from the scraper's job file, redeploy, confirm
the scraper lands on a Linux node. Sixty minutes. Straightforward.

Pre-edit inspection of the job file killed the plan in five minutes.

The scraper used the `raw_exec` driver, not a container runtime. Its
command was a hardcoded path to a Python binary at a specific
location on the workstation's filesystem. Its module path was a
hardcoded path to the project source, also workstation-specific.
These paths did not exist on any Linux node.

This was independent of the database migration. The placement pin's
load-bearing role had silently shifted. What I thought was a *data
locality* constraint was really a *host filesystem layout*
constraint — and nobody had noticed because nobody had tried to
reschedule the job since before `raw_exec` was adopted.

Three options emerged. Symlinks and distributed virtualenvs were
too fragile. Bundled-interpreter artifact stanzas had been
pre-rejected by external feedback as "often more brittle than
they look." The third option was honest: keep the pin for this
session, add the new database URL to the environment, prove
cross-node writes (compute on the workstation, data on the cluster
node), treat full containerization as the Week 3 gate that finishes
the A+ bar.

Option three won.

I added the database URL to the environment and redeployed. The
first deployment crash-looped immediately:
`ModuleNotFoundError: No module named 'rag'`.

This was the moment the skipped Week 1 checkpoint came home to
roost.

Session 4 had added a top-of-file import to the scraper module. The
launchd process on the workstation already had the right environment
variable set, so the collector kept running. The `raw_exec` job had
no such environment — but it didn't crash, because it had been
running an in-memory copy of the *pre-import-change binary* for the
last four hours. The orchestrator had never restarted the alloc.
The import bug was invisible until the first redeploy.

A second deployment added the missing environment variable. Healthy
in ten seconds. Zero restarts.

Then the cross-node write proof. With the scraper redeployed
against the new database on the cluster node, I measured a row-count
delta over 35 seconds. Twenty-seven rows — exactly three scrape
passes of nine metrics each. All nine metric series landed in
PostgreSQL on the cluster from a scraper running on the workstation.
The *data plane* was portable.

Week 2 closed with three of four A+ criteria met. The fourth
(scraper running on a non-workstation node) was blocked by an
architectural issue — hardcoded paths — that had nothing to do with
the database migration itself. Full containerization moved to
Week 3 as P0.

### The Checkpoint Lesson

Submitting the Week 1+2 consolidated external checkpoint at the end
of Week 2 forced me to confront what the missed Week 1 checkpoint
had cost: the hardcoded-paths problem was visible at Week 1 if
anyone had looked. External feedback would have flagged it. Instead
it surfaced in Session 6, costing half a session of rework and the
whole session's scope renegotiation.

External feedback was explicit about the process gap:

> "This consolidation removed the early course-correction
> opportunity. The architectural blocker could have been caught at
> Session 3 instead of reaching Session 6."

The lesson became a phase deliverable in its own right. Workflow
documentation v1.7 codified "do not consolidate checkpoints" as a
pre-handoff gate. Checkpoints are not status reports — they are
*course corrections*. A status report after the course is set is
too late.

---

## Week 3 — Containerization and Hardening

**Thesis**: Achieve computational portability and close the phase.

Session 7 was the session where the phase's shape got fixed.

The pipeline was deliberate. Deploy a private OCI registry on the
cluster. Build a multi-arch container image (both architectures for
the heterogeneous cluster) from a minimal `python:3.11-slim` base,
with the project source copied in and a standard working directory
so that path configuration becomes structurally impossible to get
wrong. Rewrite the scraper's job file: `raw_exec` becomes the
`docker` driver, the workstation placement constraint is removed
entirely, `force_pull = true` ensures the latest image is always
fetched. Rotate the database role: the initial migration's
administrative role becomes a scoped application role with CRUD-only
permissions, credential stored as an orchestrator variable and
substituted into the connection URL at deployment time.

The result came in one job status call. Allocation running on a
Linux cluster node, Healthy, writing rows to the cluster database
at normal cadence. All four A+ criteria met simultaneously. The
workstation was no longer required for any observability workload.

### Session 8 — The Infrastructure Incident

Session 7 had left one loose end. Deploying the new image required
restarting the container runtime on the data node, which forced the
orchestrator to reschedule every alloc pinned there. Most came back
healthy. Some didn't.

Four services entered permanent `pending` state.

Investigation took twenty minutes. The job files all had service
registration blocks. Service registration blocks required a service
discovery agent to register with. The data node had no such agent.
The jobs had been placed *months* earlier, before service
registration blocks became a hard dependency, and the orchestrator
had never re-evaluated them. They were running with orphaned
registration blocks — the registration calls had been silently
failing, but the workloads were fine, because the orchestrator only
evaluates registration *at placement time*.

The container runtime restart forced re-placement. Re-placement
triggered a fresh evaluation of the registration requirement. The
gap surfaced. Four jobs stuck pending.

The fix was surgical: remove the service registration blocks from
the five affected job files. These services were accessed by direct
address anyway, not through catalog lookups. Native orchestrator
service discovery was sufficient.

Jobs redeployed within minutes of the fix. All five back to healthy.

The deeper insight: **forced rescheduling is an implicit cluster
scheduling audit.** Any event that makes the orchestrator
re-evaluate placement — container runtime restart, node reboot,
leader election — is effectively a cluster-wide constraint
validation pass. Configurations that were valid at initial placement
may not be valid at rescheduling time.

Session 8 also delivered the alerting integration. A pure function
that takes a monitoring alert payload, extracts metric and node,
calls the correlation engine for Top Cause, and appends the result
as an annotation. Graceful degradation on every failure mode — the
function can add value or fail silently, but never blocks a
notification. Twenty-four tests in the suite.

Scope was deliberately logic-only. No HTTP server wrapper. External
feedback at the Session 8 checkpoint validated this scope choice
explicitly: *"Logic-only is sufficient for this phase. Adding an
HTTP wrapper is a trivial Day 2 task."* The interesting engineering
is the enrichment function; the transport layer is boilerplate.

### Session 9 — Close

Session 9 was the longest of the phase because it had to close
every loose end: zombie database audit (no orphan files requiring
archival, though one legacy file was classified and retained as a
test fallback), post-migration drift check (all services healthy
after the Session 8 fix), end-to-end data flow verification (row
counts, scraper allocation, active writes), and the
containerization of the two remaining observability jobs (the
retention manager and the log collector, using the shared image
pattern — no new Dockerfile needed).

One gotcha during the retention manager deployment: the first
deploy used the wrong orchestrator config key (the one that
overrides the container's default command, not the one that
overrides its entrypoint). The image's entrypoint was the scraper,
so the retention manager allocation was running the scraper with
retention args. Five-minute fix, documented for future
containerization sessions.

Two ADRs written in this session. ADR-018 documented the PostgreSQL
migration decision — the connection factory, the Python-side cutoff
strategy, the big-bang cutover with environment-variable rollback.
ADR-019 documented the alerting integration — the webhook
enrichment pattern, the graceful degradation contract, the
deliberate logic-only scope.

The third ADR from this phase, ADR-020, was the architectural
novelty. It did not come from a planned design session. It came
from the Session 8 incident. Two decisions:

- **Decision 1**: Use native orchestrator service discovery on
  nodes where the service discovery agent isn't installed. Remove
  registration blocks from affected jobs.
- **Decision 2 [PRIMARY]**: Use job-file constraints as durable
  placement policy. The exclusion constraint is committed to git;
  node eligibility flags are ephemeral operational state that can
  be silently reversed.

The Decision 2 insight — the distinction between *operational
state* and *architectural intent* — was the architectural novelty
of the phase. Two mechanisms exist for controlling job placement in
the orchestrator. They serve fundamentally different purposes:

| Mechanism                    | Type                  | Scope         | Durability |
|------------------------------|-----------------------|---------------|------------|
| Node eligibility command     | Operational state     | Cluster-wide  | Ephemeral  |
| Job-file constraint block    | Architectural intent  | Job-specific  | Durable    |

Operational state answers: "Is this node currently accepting work?"
Architectural intent answers: "Should this specific job ever run on
this node?" These are different questions. Encoding architectural
intent as ephemeral operational state creates a time bomb: it works
until someone resets the ephemeral state. The pattern generalizes
beyond this orchestrator — any system where "where does this run?"
is answered by mutable runtime state has the same class of drift
vulnerability.

All nine phase sessions closed. External validation at Session 9
confirmed:

> "The A+ bar is officially cleared. You have achieved true
> Computational Portability. A powerful signal of professional
> system design."

---

## Lessons Learned

### The Checkpoint Cadence Lesson

Skipping Week 1's checkpoint cost half a session of rework and a
full session of scope renegotiation. At the checkpoint's ~20-minute
time budget, that math never comes out in favor of skipping.
Checkpoints are course corrections, not status reports.

### The Python-Side Cutoff Lesson

The most impactful line of the migration was a three-line cutoff
computation in Python. When two SQL dialects diverge on *computing
a value*, compute it in application code instead. Pass it as a
parameter. Both dialects accept parameters identically. The
dialect-neutral language is the host language, not the database.

### The Zombie Database Pattern

After a migration, the old files are a *temptation*, not just
clutter. A developer (or an AI assistant) might connect to the
deprecated local file and never know they're not writing to
production. Every migration produces zombies. Schedule the audit as
a deliberate post-migration step.

### The Multi-Tenant Preflight Lesson

"Provision X" in a task plan is always suspect. Check whether X
already exists before provisioning. The cost of the check is one
command; the cost of missing it can be catastrophic for unrelated
services sharing the same host.

### The Forced-Rescheduling Audit Lesson

Any event that forces a scheduler to re-evaluate placement is
effectively a cluster-wide constraint validation pass.
Configurations that were valid at placement time may not be valid
at rescheduling time. This led to the **Infrastructure Blast
Radius Checklist** in the workflow documentation: before any
operation that could force rescheduling, enumerate all dependent
services and verify their scheduling requirements are still
satisfied.

### The Operational vs Architectural Lesson

Two mechanisms can control the same thing, and still serve
different purposes. Placement policy is an architectural decision;
encoding it as ephemeral operational state creates a time bomb.
The durable mechanism (version-controlled, job-specific,
git-traceable) wins every time for architectural intent. The
ephemeral mechanism is useful as a complementary layer, not a
substitute.

---

## What This Phase Proves

Phase 21 is the first phase whose portfolio story is *not* about
intelligence. It's about operational maturity. Database migration
with zero downtime. Multi-arch containerization. Least-privilege
IAM. Distributed compute. Infrastructure-as-code. Architectural
decision records capturing both planned decisions and unplanned
insights.

The methodology hallmarks held: hypothesis-driven Session 0 planning
with external validation, weekly checkpoint cadence (enforced hard
after the checkpoint lesson), zero-regression standard, two-version
documentation, portfolio sync per phase.

The system is no longer shaped like a single-node project. The
development workstation runs the UI and the language model, and
nothing else from the cluster side. The three Linux nodes carry
every observability workload. PostgreSQL is the single source of
truth. The scraper writes across the network. The correlation
engine reads from the cluster database. The alerting module
enriches monitoring notifications with reasoning that came out of
the same correlation engine the UI uses.

External validation called it **"true Computational Portability."**
My phrase for it is **"the shape finally matches the claim."**

---

**Phase 21 — Distributed Foundations. Complete.**
