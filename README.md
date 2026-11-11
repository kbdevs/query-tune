<p align="center">
  <img src="docs/query-improvements.png" alt="DB queries take up to 99% less time. Memory save batch: Before 543.00 ms, After 25.20 ms; Library project totals: Before 144.58 ms, After 119.85 ms; Library chat totals: Before 133.08 ms, After 110.61 ms; Latest scheduled run: Before 60.20 ms, After 2.82 ms; Branched chat history: Before 59.20 ms, After 2.17 ms; Memory wipe batch: Before 38.40 ms, After 13.70 ms; Sharing badges: Before 31.80 ms, After 0.81 ms; Share check: Before 30.60 ms, After 0.44 ms; Email lookup: Before 17.50 ms, After 0.23 ms; Image key check: Before 13.50 ms, After 0.25 ms; Memory recovery queue: Before 9.78 ms, After 2.05 ms." width="820">
</p>

<h1 align="center">query-tune</h1>

<p align="center">
  An agent skill to find avoidable database work,<br>
  preserve results, and measure the change.
</p>

<p align="center"><sub>Recorded before-and-after results. Read <a href="#what-the-numbers-mean">what the numbers mean</a> before quoting them.</sub></p>

## What's in it

| Skill | What your agent does with it |
| --- | --- |
| [`query-tune`](skills/query-tune/SKILL.md) | Traces the caller and query plan, finds repeated or unbounded work, preserves results and locks, and measures the tradeoffs. Audits start read-only. |

The skill is plain instructions; it does not require a particular database client or application schema. Use the tools and access rules of your project. The recorded examples use MySQL; plan syntax, collation, locking, and index behavior differ between engines.

## Install

Clone the repo:

```bash
git clone https://github.com/kbdevs/query-tune.git
```

For Codex, copy the skill into its user skills directory:

```bash
mkdir -p ~/.codex/skills && cp -R query-tune/skills/query-tune ~/.codex/skills/
```

For Claude Code:

```bash
mkdir -p ~/.claude/skills && cp -R query-tune/skills/query-tune ~/.claude/skills/
```

For one project, copy the skill folder into its `.agents/skills/` or `.claude/skills/` directory. Check your agent's configuration if it uses a different skills directory. Review an existing skill before replacing it.

## Try it on your database

Paste this into your agent. It reads only; nothing changes until you say so.

```text
Use the query-tune skill. Audit this project's database queries in read-only mode.
1. Read the source, schema definitions, indexes, and available query digests or saved plans. Use only access already permitted for this project. Do not collect credentials or row content.
2. Rank the expensive paths by total time, rows examined, statement count, and lock waits where evidence exists. State what is unknown instead of guessing.
3. Trace the callers of the strongest candidates. Explain the cause, the smallest proposed change, the result and locking rules it must preserve, and the evidence needed to test it.
4. List separately any schema change, extra write cost, changed result contract, or production action.
Report first. Do not change files, data, schema, settings, or infrastructure. Do not run benchmarks, locking queries, or plan commands that execute a query. Wait for my instruction before making any change.
```

## What query-tune covers

- **Access paths.** Missing composite-key columns, functions on indexed filters, state ranges, and latest-per-parent reads.
- **Repeated work.** Per-row lookups and writes, duplicate state changes, and reads beyond the caller's page or history window.
- **Correctness.** Tenant boundaries, exact owner/key pairs, collation, ordering, cursors, visibility, retries, and required locks.
- **Measurement.** Pinned before and after code, realistic small and large profiles, randomized pairs, errors, p95, statements, and lock costs.
- **Tradeoffs.** Small-case regressions, added statements, write and storage cost, fallback work, and commit cost excluded by rollback benchmarks.

## What the numbers mean

The hero chart shows 11 selected query and batch paths from separate local studies. It is an example of recorded improvements, not a prediction for your database.

- Local MySQL, eight clients, 50 paired blocks per cell, synthetic profiles.
- Selected benchmarks, not every workload. Small cases can regress.
- Production latency was not measured. No end-to-end app speed or cost savings are claimed.
- Compare Before and After within a row. Rows use different datasets and workloads; do not add the savings into a total.
- Batch timings exclude commit cost because the benchmark rolls back writes.
- Both dense Library cases remain above the study's 100 ms budget.
- The scheduled-run profile exceeds the retention cap. Its rewrite also fixes omitted older runs, so it is not solely a performance change.
- The branched-history case uses 10,000 messages and no files. A full-read fallback remains.
- Chart labels round to two decimals; percentages round to whole numbers from the source medians. Those percentages are not the studies' statistical paired-ratio estimates.

| Path | Before (ms) | After (ms) |
| --- | ---: | ---: |
| Memory save batch | 543 | 25.2 |
| Library project totals | 144.575 | 119.851 |
| Library chat totals | 133.075 | 110.605 |
| Latest scheduled run | 60.2 | 2.82 |
| Branched chat history | 59.2 | 2.17 |
| Memory wipe batch | 38.4 | 13.7 |
| Sharing badges | 31.8 | 0.809 |
| Share check | 30.6 | 0.435 |
| Email lookup | 17.5 | 0.234 |
| Image key check | 13.5 | 0.248 |
| Memory recovery queue | 9.78 | 2.05 |

The private application code and raw benchmark samples are not included. This repo alone cannot reproduce the timing studies. No new timing benchmark was run to make this repo.

## Layout

```text
skills/
  query-tune/
    SKILL.md
docs/
  query-improvements.png
LICENSE
README.md
```

## License

[MIT](LICENSE)
