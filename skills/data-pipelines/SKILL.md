---
name: data-pipelines
description: Use when designing, building, or debugging ETL/ELT jobs, data pipelines, batch imports/exports, scheduled data jobs, backfills, CDC, or stream processing — idempotent re-runs, watermarks, schema drift, and data-quality gates.
---

# Data Pipelines

Pipeline correctness = what happens on the re-run. Design for re-run, backfill, and partial failure first; the happy path is the easy 20%. A pipeline that can't be safely re-run at 3am isn't done.

## Idempotency — the core property

Every run must be re-runnable without duplicating or corrupting data. Pick one mechanism and state it:

- **Upsert/merge** on a natural key,
- **Delete + insert the partition** (day/batch) atomically,
- **Staging table + swap** for full replaces.

Append-only writes + any retry = duplicates, guaranteed. If dedup happens downstream "later", it happens never.

## Incremental loads

- Watermark/cursor on a **reliable monotonic column** — `updated_at` needs a late-arrival overlap window (re-read the last N minutes/hours); auto-increment IDs miss updates entirely. Real change capture → CDC.
- Persist the watermark transactionally WITH the load — watermark advanced but load failed = silently lost rows forever.
- Keep the full-reload path working and tested; it's the recovery story when the incremental logic is ever wrong.

## Backfills

Parameterize by date-range/partition from day one, and make backfill **the same code path** as the scheduled run — a hand-edited one-off script drifts from production logic and reintroduces the bug you're backfilling away. Big backfills run in bounded chunks (memory + lock time) and are resumable per chunk.

## Partial failure

- Unit of atomicity = partition/batch, in a transaction: a crash mid-run leaves loaded partitions consistent and unloaded ones absent — never half a partition.
- Bad rows: don't fail a million-row load for three malformed rows — divert them to a dead-letter/reject table **with counts alerted**. Silently dropping them is data loss with extra steps.
- Retries, timeouts, poison-message handling on sources/sinks → `error-resilience`. Parallel partition loads → `concurrency-async`.

## Data-quality gates (end of every run)

Row count vs source (or vs expected delta) · uniqueness/null checks on keys · **freshness check** ("newest row < X hours old"). Gates fail loud. Crucially: alert on the run NOT happening (missed schedule, zero rows) — the deadliest pipeline failure is the silent no-op → `observability`.

## Schema drift

Ingestion has an explicit contract (schema validation, typed staging table). New/unknown upstream columns are a logged decision — map it or consciously ignore it — never a silent drop. Renamed/retyped source columns must fail the run, not load garbage. Target schema changes → `db-and-migrations` (expand-contract applies to warehouses too).

## Orchestration

Small scale: cron + a run-lock + structured logs covers a handful of jobs; reach for an orchestrator (Airflow/Dagster/Temporal) only when real DAG dependencies, per-task retries, and a backfill UI earn their ops cost. What's never fine: overlapping runs unguarded (lock or skip), and job B assuming job A finished because "it usually has" — encode the dependency or check the freshness gate.

## Checklist

Re-run mechanism named (upsert / partition-replace / swap) · watermark transactional with load + late-arrival window · backfill = same code path, chunked · partial failure leaves consistent partitions · rejects counted + alerted, not dropped · quality gates incl. freshness + missed-run alert · schema contract explicit · overlap-guarded scheduling.
