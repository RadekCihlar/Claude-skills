---
name: performance-profiling
description: Use when something is slow — diagnosing latency/throughput problems in existing code, profiling, query analysis, load testing, memory leaks. Complements security-perf-preflight (which is for code being written).
---

# Performance Profiling

Rule one: **measure before touching anything.** Intuition about hotspots is wrong more often than right, and an optimization without a before/after number is a rumor.

## Frame the problem as a number

"It's slow" → "checkout p95 is 3.1s, target 800ms, worst at peak traffic". Percentiles, not averages — the mean hides the pain (p99 is where real users suffer). Know which resource is saturated: CPU-bound, I/O-wait, lock contention, GC — the fix differs completely.

## Measure with the right instrument

- **Distributed/request latency** → traces or timing logs per segment: where do the 3.1s actually go? Usually 80% sits in one hop.
- **CPU** → sampling profiler / flamegraph (JVM: async-profiler, JFR; Node: `--cpu-prof`/clinic; Python: py-spy). Widest tower = the target.
- **DB** → slow-query log + `EXPLAIN (ANALYZE, BUFFERS)` on real data volume — dev-sized tables lie; seq scan on 1k rows is invisible, on 50M it's the outage.
- **Memory** → heap snapshots/dumps compared over time for leaks; allocation profiler for churn/GC pressure.
- JVM specifics: measure after warmup (JIT), or you're profiling the compiler.

## Where it usually is (check in this order)

1. **The database**: N+1 query loops, missing index, over-fetching (`SELECT *`, loading relations you don't use), no pagination.
2. **Chatty I/O**: sequential awaits that could batch or parallelize; per-item HTTP calls that should be one bulk call.
3. **Sync work blocking the hot path**: file/network I/O, crypto, compression on the request thread.
4. **Allocation churn**: large objects built and discarded per request; string concat in loops; GC pauses following.
5. **Locks/contention**: threads waiting, not working — shows as low CPU + high latency.
6. Only then micro-stuff — and usually never.

## Fix in order of leverage

Do less work (algorithm, drop the query, batch) → do it elsewhere (async, background, precompute) → do it once (cache — see caching-strategy) → do it faster (micro-opt, last and rarest). After EACH change: re-measure the same number, keep the receipt (before/after in the PR). No proof, no merge.

## Load & regression testing

Test with realistic data volume AND concurrency; find the knee (where latency departs linearity), not just the happy load. Soak test (hours) for leaks and drift. Guard wins with a budget: a perf test in CI or a p95 alert (observability) — otherwise the regression returns silently in three sprints.

## Frontend

Same discipline: Lighthouse/DevTools performance panel, network waterfall. Usual suspects: bundle size (analyze, code-split), render-blocking resources, unoptimized images, layout thrash, main-thread long tasks. Budgets = CWV (see seo-technical).

## Checklist

Number defined (percentile + target) · profiled, not guessed · DB checked first · fix at highest leverage rung · before/after measured + recorded · regression guard added.
