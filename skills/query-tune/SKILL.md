---
name: query-tune
description: Diagnose and improve slow SQL queries using execution plans, existing indexes, bounded reads, and paired benchmarks. Use for query latency, excess statements, or lock contention; follow the project's migration process for schema changes.
---

# Query tune

Find the work the database does that the caller does not need. Change the
smallest shared query path that removes it. Keep the caller's result contract.

## Start read-only

For an audit, read source, schema definitions, saved plans, and permitted query
metadata. Report findings before changing anything. Do not edit files, change
data or schema, run load tests, or execute queries that acquire locks until the
user authorizes that work. Use saved evidence or a plan-only command when live
execution is not authorized.

## Find the cause

Trace the caller, all uses of the query, its transaction, and the indexes that
exist. Record result order, page cursors, tenant scope, visibility rules,
collation, NULL behavior, and retry rules. Separate essential locks and writes
from advisory reads and duplicate state.

Use query digests to rank total time, rows examined, call count, and lock waits.
A frequent small query can cost more than one slow query. Use the project's
approved read-only access for production evidence. Do not collect row content
or credentials. Run ANALYZE or locking statements only where execution is
authorized; an execution plan can execute the query.

Read the plan against realistic selectivity. Record access path, index columns
used, rows read, sorts, table lookups, and where LIMIT takes effect. A LIMIT
does not bound the scan or the locks. FOR UPDATE SKIP LOCKED can lock rows it
scans while finding a result.

## Prefer the existing access path

Apply only the mechanism supported by the plan:

- Supply missing leading columns of a composite index. Keep tenant scope in
  each branch. Use an existing key hash for lookup and the full key to reject
  hash collisions.
- Avoid functions on indexed filter columns when their collation already gives
  the required comparison. Prove equality under the actual database collation.
- For an ordered page across states, consider one ordered, limited range per
  state, then merge those pages. Use one UNION ALL statement when one snapshot
  is required. Separate reads can miss or duplicate rows that change state.
- For the latest row per parent, probe each bounded parent range instead of
  sorting its entire history. For OR across different indexes, consider
  separately bounded probes. Count the extra statements.
- Batch repeated reads and writes. Keep exact (owner, id) pairs; two independent
  IN lists can admit the wrong combinations. Preserve deduplication, per-row
  generations, consent checks, and mutation locks. Let SQL decide equality when
  application string comparison differs from the column's collation.
- Read only the page or history window the caller needs. Prove that skipped
  rows cannot affect it. Keep a correct fallback for incomplete paths. Test the
  fallback cost as well as the fast case.
- Remove duplicate state writes only after checking every reader, replay gate,
  foreign key, and downstream event. Keep required persistence and locking.

Do not add a cache or index before checking these options. Use an index hint
only when a measured optimizer error warrants it; test small and large shapes.
When an index is needed, measure write and storage cost and follow the
project's schema review and rollout rules. A local index experiment does not
authorize a production migration.

## Prove the result and measure the tradeoff

Compare old and new results on the same fixtures before timing. Cover ties,
cursor boundaries, missing rows, duplicate inputs, case and accent comparisons,
tenant isolation, and relevant concurrent changes. For claims or writes, also
compare effects and locks under two sessions.

Vary relationship fanout separately from population. Include empty and small
controls, product caps, skew, large histories, and more than one client. A
synthetic multiplier is not a production volume. Seed unrelated owners so an
owner does not accidentally hold most of every table.

Pin both implementations and randomize paired before/after blocks in the same
environment. Keep host load, warmup, engine settings, and concurrency in the
record. Use enough pairs for the desired inference; label short runs as smoke
checks. Report medians, p95, errors, statement counts, rows examined, and lock
counts when relevant. Use confidence intervals and a multiple-comparison
correction when drawing conclusions across many cells. Do not treat a quotient
of aggregate medians as the geometric mean of paired ratios.

For rolled-back write benchmarks, say that commit cost is excluded. Check
whether rollback changes optimizer statistics. Keep raw samples, result
signatures, plans, source revisions, and reproduction settings.

## Deliver

Give the root cause, the change, the evidence, small-case regressions, and the
remaining limits. Separate observed production results, local measurements,
plan estimates, and experiments. A merged change does not establish production
deployment or a measured user-visible gain.

For a public graph, use a few comparable before/after bars, a zero baseline,
units, exact values, and measurement context. Keep experiments out of a chart
labelled as merged improvements. Include a source table. Do not infer cost
savings or end-to-end speed from query timing alone.
