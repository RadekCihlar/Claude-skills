---
name: observability
description: Use when adding or reviewing logging, metrics, tracing, or alerts — what to log, at what level, structured fields, cardinality limits, and making failures diagnosable from telemetry alone.
---

# Observability

The test: can you diagnose a production incident from telemetry alone, without adding a log line and redeploying? Instrument for that reader.

## Logging

- **Structured, not prose.** Key-value fields the platform can filter (`{event: "payment_failed", order_id, upstream_status, duration_ms}`), message as a stable short string. No string-interpolated novels — they can't be queried.
- **Log at boundaries and decisions**, not every line: request in/out (with duration + status), external calls that fail or exceed a threshold, state transitions, every caught error WITH its context, every retry/fallback taken. Loop bodies and per-item logs on hot paths = noise + cost.
- **Levels mean something.** `error` = broken, someone may need to act (alertable). `warn` = survived but abnormal (fallback used, retry succeeded, deprecated path hit). `info` = state changes worth an audit trail. `debug` = development detail, off in prod by default. If everything's `error`, nothing is.
- **Never log**: secrets, tokens, passwords, full auth headers, card numbers, raw PII (emails/names → mask or use IDs). This is a security boundary, same rank as SQL injection.
- **Correlation ID everywhere.** Generate/propagate a request/trace ID through every log line, queue message, and downstream call header — the difference between a story and confetti.
- Match the project's existing logger and field conventions; don't introduce a second logging style.

## Metrics

- **RED for services** (rate, errors, duration — histograms, not averages; p95/p99 is where users live). **USE for resources** (utilization, saturation, errors).
- **Cardinality is the budget.** Labels are bounded sets (route template, status class, region) — NEVER user IDs, raw URLs, emails, or anything unbounded; one bad label can take down the metrics stack. High-cardinality questions belong to traces/logs.
- Instrument the four moments: request handled, external call made, job processed, queue depth. Business counters (`orders_created_total`) only for numbers someone will actually watch.

## Tracing

If the platform has tracing, propagate context (W3C traceparent) through HTTP calls and queue messages; span per external call with error status recorded. Don't hand-roll spans around trivial in-process functions.

## Alerts

Alert on symptoms users feel (error rate, p99 latency, queue lag, saturation), not causes (CPU%). Every alert has: a threshold with a reason, a runbook line ("check X, restart Y"), and an owner. An alert that pages with no action attached gets deleted — pager fatigue is how real incidents get ignored.

## Checklist

Structured fields · correct level · no secrets/PII · correlation ID propagated · failures logged with context at the boundary · metric labels bounded · alerts actionable.
