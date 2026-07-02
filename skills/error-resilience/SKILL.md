---
name: error-resilience
description: Use when writing code that calls external services, does I/O, processes jobs/queues, or handles failures — timeouts, retries, fallbacks, idempotency, and error messages that actually help at 3am.
---

# Error Handling & Resilience

Everything remote fails: slowly, partially, and at the worst time. Design for the failure, not just the happy path.

## The non-negotiables

- **Timeout on every external call.** No unbounded waits, ever — network default is "hang forever". Pick a number from the caller's budget (user-facing request ≈ seconds, batch job ≈ more), pass it explicitly.
- **Retry only what's safe to retry.** Idempotent operations (GET, PUT with same body, POST with idempotency key) → retry on transient errors (timeout, 429, 5xx, connection reset). Non-idempotent without an idempotency key → do NOT blind-retry; you'll double-charge someone.
- **Backoff + jitter + cap.** Exponential backoff with jitter (avoid thundering herd), max attempts (2-3 is usually right), total-time cap inside the caller's budget. Honor `Retry-After` on 429.
- **Fail fast at the edge, degrade in the middle.** Invalid input → reject immediately with a clear message. Optional downstream (recommendations, avatars, enrichment) failing → serve the core response without it, log the gap. Core dependency down → clear error, not a hang.
- **No silent failures.** Empty catch, bare except, catch-that-only-logs while pretending success, `.catch(() => {})` — all banned. Handle it meaningfully, rethrow with context, or let it crash loudly.

## Error design

- **Two audiences, two messages.** Outward: what failed + what the caller can do, no internals, no stack, no secrets. Inward (logs): full context — operation, inputs (sans secrets/PII), upstream response, correlation/request ID.
- **Wrap with context as errors cross layers**: `fetch config failed: GET https://… : connection refused` — cause chain preserved (`cause` option / `from err`), so the 3am reader gets the whole story from one log line.
- **Expected vs exceptional.** Expected outcomes (not found, validation failed, conflict) are modeled results/typed errors the caller must handle; exceptional ones (bug, dependency down) propagate to a boundary handler. Don't use exceptions as control flow; don't return `null` where an error belongs.
- One top-level boundary per surface (request handler wrapper, job runner, main) that catches, logs with context, and converts to the surface's error shape — instead of try/catch confetti at every layer.

## Systemic patterns (reach for when the blast radius justifies)

- **Circuit breaker** when a flaky dependency can eat your thread/connection pool: trip after repeated failures, fail fast while open, probe half-open. Use an existing lib, don't hand-roll.
- **Queues/jobs**: at-least-once delivery → handlers idempotent (dedupe key), poison messages go to a dead-letter after N attempts instead of blocking the queue forever.
- **Graceful shutdown**: stop intake, finish in-flight (bounded), then exit — or your deploys create the very errors this file prevents.

## Checklist

Every external call has timeout · retries only on idempotent + transient, with backoff/jitter/cap · optional features degrade, core fails clearly · no swallowed errors · outward messages clean, inward logs complete with correlation ID · job handlers idempotent.
