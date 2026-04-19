# ADR-018: PostgreSQL Migration — Distributed Data Plane

**Status**: Accepted
**Date**: 2026-04-12
**Phase**: 21 — Distributed Foundations
**Deciders**: Michel Mazza

---

## Context

The previous cluster architecture kept its observability database (a
SQLite file) on the development workstation's local filesystem. Every
job that wrote to it — the Prometheus scraper, the log event
collector, the retention manager — inherited a placement constraint
pinning it to that one host. This was the **"single-node pin"**: a
coupling between compute and data that foreclosed distributed
scheduling.

The pin had two failure modes. Low-probability / high-impact: the
workstation's disk is the single point of data. High-probability /
low-impact: every new observability feature inherited the pin as a
precondition, and so every feature was harder to deploy to a
distributed runtime.

Phase 21 was scoped to break the data coupling. Once the database
lived on a network-accessible host, any cluster node could run the
scraper — and the broader architectural shift from single-node
development to distributed production could begin.

---

## Decision

**Migrate the SQLite database to PostgreSQL on a dedicated Linux node,
via a backend-agnostic connection factory, with a big-bang cutover
strategy.**

### The Connection Factory

The centerpiece is a 137-line module (`rag/db.py`) that routes every
consumer's database access through a single factory function. The
signal for routing is the URL scheme:

```
postgresql://...  -> Psycopg 3 connection (lazy import)
sqlite:///...     -> internal wrapper with dialect translation
empty / unset     -> default SQLite (tests and development)
unknown scheme    -> ValueError (fails loudly)
```

There are no environment-based fallbacks and no "try PostgreSQL, fall
back to SQLite on error" logic. Silent fallbacks hide
misconfigurations; explicit errors force the caller to declare intent.

### Transparent Dialect Translation

Consumer modules write Psycopg-native SQL with `%s` placeholders. An
internal `_SQLiteWrapper` translates to SQLite's `?` on every
`execute()` call. This means the codebase has exactly one SQL dialect
in its consumer code, even though SQLite remains the default for tests
and development. The dialect-branching surface area is one line,
inside one wrapper.

The wrapper also enables WAL mode on every SQLite connection and
provides context-manager semantics (commit on successful exit,
rollback on exception) that match PostgreSQL's behavior. Consumers
write idiomatic Python (`with conn: ...`) and the backend-specific
transaction management is invisible.

### Python-Side Timestamp Cutoffs

Temporal queries were the largest source of dialect divergence.
SQLite's `strftime('%s', 'now') - ?` and PostgreSQL's
`EXTRACT(EPOCH FROM NOW()) - %s` cannot coexist in the same string.

The resolution: **compute the cutoff in Python, pass it as a bound
parameter.**

```
# Before (SQLite-specific)
"DELETE FROM metric_snapshots WHERE timestamp < strftime('%s','now') - ?"

# After (backend-agnostic)
cutoff = int(time.time()) - seconds
"DELETE FROM metric_snapshots WHERE timestamp < %s"
```

Both backends receive the same SQL. The dialect surface area for
temporal queries drops to zero. This was the single most impactful
architectural decision for backend portability — it eliminated the
largest class of branching the codebase would have otherwise needed.

### Minimum-Surface Dialect Gate

For the rare cases where SQL expressions genuinely differ (JSON
extraction being the main one), the factory exposes an
`is_sqlite_connection()` predicate. Callers use it for one or two
narrow branches. This is the minimum-surface-area dialect gate: it
keeps the wrapper internal while exposing the one predicate callers
actually need.

### Big-Bang Cutover

The migration strategy was a single-cutover swap, not dual-write or
phased rollout:

1. Provision PostgreSQL, migrate schema and data. SQLite remains
   authoritative.
2. Port each consumer module to the factory. SQLite is still the
   default; PostgreSQL activates only when the env var is set.
3. Flip the env var. PostgreSQL becomes authoritative; SQLite becomes
   the test/dev default.

The rollback plan is a single environment variable flip. SQLite data
remains on disk throughout the migration window, so a revert loses at
most the minutes of writes that went to PostgreSQL after the flip.

Dual-write was considered and rejected. It would have required two
backends to be simultaneously authoritative and would have complicated
every consumer path. For observability data (continuously replaced
time-series with a rolling retention window), dual-write adds
significant complexity for no operational gain.

---

## Consequences

### Positive

- **Compute is no longer coupled to a single node**: the scraper runs
  on any eligible cluster node and writes to the centralized database.
  The architectural smell is removed.
- **Single SQL dialect in consumer code**: all new code is written in
  Psycopg conventions; the wrapper handles SQLite translation
  transparently. Future developers do not have to learn two dialects.
- **Backend-agnostic retention**: the retention manager became the
  sole authority for both backends. The split-brain "two retention
  paths, one per backend" story is eliminated.
- **Explicit environment declaration**: the `DB_URL` env var is the
  single truth for backend selection. Tests default to the embedded
  database. Production sets PostgreSQL. There is no ambiguity about
  which backend is active in any given context.

### Negative

- **Network dependency**: the scraper now requires a working network
  path to the database host. Acceptable — the same host serves other
  critical homelab workloads whose availability posture is already a
  first-class operational concern.
- **Two database backends in the codebase**: SQLite remains
  load-bearing for tests. Any new SQL has to be written in PostgreSQL
  conventions, which means occasional debugging when a SQLite quirk
  surfaces (JSON extraction, specifically).
- **Rollback is cheap but not automatic**: a single env var flip
  reverts the scraper to SQLite, but data will have diverged during
  the live window. Acceptable because observability data is
  continuously replaced.

### Neutral

- **IAM rotation was a separate step**: the initial cutover used a
  SUPERUSER role for velocity. A subsequent session rotated to a
  scoped application role with CRUD-only permissions on the
  observability tables, stored as a secret and substituted into the
  connection URL at job deployment time.

---

## Alternatives Considered

### NFS Mount for SQLite

Rejected. Adds a new single point of failure (the NFS server) and
SQLite's WAL mode has known issues on network filesystems. Does not
solve the real problem — just makes more nodes depend on the
workstation.

### Litestream (Replicated SQLite)

Rejected. Still has a single-writer constraint. The writer would have
remained the workstation — no change to the placement pin. Read
replicas would be eventually consistent, which complicates
correlation queries that depend on strict event ordering.

### Dual-Write Phased Migration

Rejected. Four-plus additional sessions of complexity. Every consumer
needs dual-write logic during the transition. Observability data is
continuously replaced; historical correctness is not load-bearing for
a system with a rolling retention window.

### SQLAlchemy Core

Considered and deferred. Would work. Heavier dependency than needed
for the migration scope — the custom factory is 137 lines, whereas
SQLAlchemy Core would have added a framework to learn and debug. If a
future phase needs ORM-style querying or more sophisticated schema
management, SQLAlchemy Core becomes viable.

---

## Lessons Learned

### Python-Side Cutoffs vs SQL-Side Arithmetic

The most impactful decision in the migration was computing timestamp
cutoffs in Python and passing them as bound parameters. It eliminated
the largest class of dialect branching the codebase would have
otherwise needed.

**Generalization**: When two SQL dialects diverge on *computing a
value*, compute it in application code instead. Pass it as a
parameter. Both dialects accept parameters identically.

### The "Zombie SQLite" Pattern

After a database migration, legacy local files become a *temptation*,
not just clutter. Developers (or AI assistants) could accidentally
connect to the deprecated local file instead of the live production
database, leading to silent configuration drift.

**Generalization**: After a database migration, audit the legacy
files. Either delete them (destructive but clear) or rename them
(`*.deprecated`) to make accidental connection impossible.

### Latent Import Bugs in Long-Running Orchestrated Processes

During the migration, a top-of-file import was added to a service
module. The service did not restart — the running process had already
imported its pre-change binary into memory. The bug was silent for
48 hours until a redeploy triggered a fresh import and a
`ModuleNotFoundError` appeared.

**Generalization**: Changes to top-of-file imports in modules managed
by non-restarting orchestrators are time-bombs until the next forced
restart. Containerization eliminates this class of bug: a standardized
working directory with the module tree copied in makes path
configuration structurally impossible to get wrong.

### Multi-Tenant Database Preflight

"Provision X" in a task plan is always suspect. Check whether X
already exists before provisioning. The cost of the check is one
command; the cost of missing it can be catastrophic for unrelated
services sharing the same host.

---

**ADR-018 Status**: Accepted (2026-04-12)
**Supersedes**: None
**Superseded by**: None
