---
name: db-and-migrations
description: Use when designing database schema, writing non-trivial queries, or creating migrations — constraints, indexing, N+1, transactions, and zero-downtime migration safety (expand-contract).
---

# Database & Migrations

The database outlives the app code. Integrity lives in the schema, not in app-level promises.

## Schema design

- **Constraints in the DB.** NOT NULL, UNIQUE, FK, CHECK — the DB enforces what must always be true; app validation is UX, not integrity. A "we validate in the app" invariant is already broken somewhere.
- **Types that mean it.** Money → integer cents or DECIMAL, never float. Time → UTC `timestamptz`. IDs → pick one strategy (bigint identity or UUIDv7) and stay consistent. Enums → CHECK or lookup table over magic strings.
- **Name boringly and consistently** with the existing schema: `snake_case`, singular/plural — match what's there.
- Soft-delete only when the product needs undelete/audit; it complicates every query and unique index (`WHERE deleted_at IS NULL` partial indexes).

## Queries

- **N+1 is a design bug, fix at write time.** Query inside a loop → join, `IN` batch, or dataloader now.
- **SELECT what you use**, not `*`, on hot paths and wide tables.
- **EXPLAIN anything non-obvious** on realistic data volume; dev-sized tables hide every problem.
- Pagination: keyset/cursor (`WHERE (created_at, id) < (?, ?) ORDER BY … LIMIT ?`) over OFFSET for deep pages.
- Lock discipline: keep transactions short; no network calls inside a transaction; consistent table-access order to avoid deadlocks; `SELECT … FOR UPDATE` only with a reason.

## Indexes

- Index what queries filter/join/sort on — derived from real query patterns, not guessed.
- Composite index column order: equality columns first, then range/sort. An index on `(a, b)` serves `a` and `a,b`, not `b` alone.
- Every index taxes writes; don't shotgun. Unique constraints double as indexes.
- Foreign keys usually want an index on the referencing column (FK alone doesn't create one in Postgres).

## Migrations — zero-downtime rules

Old code and new schema (and vice versa) WILL run together during deploy. Therefore:

- **Expand → migrate → contract.** Add nullable column / new table first; deploy code writing both / reading either; backfill; then in a LATER release make it required / drop the old thing. Never rename in place — add new, migrate, drop old later.
- **Destructive steps ship separately** from the code that stops using the thing, at least one release apart. Dropping a column/table the previous version still reads = outage.
- **No long locks.** Postgres: `CREATE INDEX CONCURRENTLY`; add column without volatile default on big tables (or on modern PG, non-volatile default is fine); `NOT NULL` via `CHECK … NOT VALID` + `VALIDATE`. Big backfills run in batches with sleep, not one UPDATE.
- **Reversible or explicitly not.** Every migration states its rollback; irreversible ones say so and require a backup checkpoint first.
- Migrations are append-only history: never edit an applied migration; fix forward with a new one.

## Red flags — thoughts that mean STOP

| Thought | Reality |
|---|---|
| "Small table, direct ALTER is fine" | Prod size ≠ dev size; locks queue behind long transactions. Safe pattern costs one extra step. |
| "Old code is gone right after deploy" | Old and new run together during rollout — and rollback brings old back. Expand-contract. |
| "We'll drop the old column later" | Later = never unless the contract step is a scheduled release. Ticket it now. |
| "The app validates this, no constraint needed" | Every app bug, script, and manual fix bypasses app validation. The DB is the last line. |
| "This backfill is quick" | One UPDATE over millions of rows = long lock + replication lag. Batch with sleeps. |
| "I'll just edit the old migration" | Applied migrations are history; editing desyncs every environment. Fix forward. |

## Checklist

Constraints enforce the invariant · types exact (money/time/id) · no query-in-loop · indexes match real predicates · migration safe with old code still running · destructive change deferred a release · rollback stated.
