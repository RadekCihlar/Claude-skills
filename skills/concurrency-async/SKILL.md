---
name: concurrency-async
description: Use when writing code with shared state, parallel work, threads/coroutines/async-await, background jobs, or diagnosing race conditions, deadlocks, and "works locally, corrupts in prod" bugs.
---

# Concurrency & Async

Concurrency bugs don't fail tests — they fail at 3am under load. Design them out structurally; you cannot debug your way to thread safety.

## The root of all races: shared mutable state

Prefer, in order: immutable data → state confined to one owner (one thread/actor/coroutine owns it, others message) → synchronized access as last resort. If two writers can touch the same thing, assume they eventually will simultaneously.

## Race patterns and their fixes

- **Check-then-act** (`if not exists → create`, `if balance > x → withdraw`): the gap between check and act is the bug. Fix with an atomic operation at the source of truth: DB unique constraint + handle the conflict, upsert, atomic compare-and-swap — not a bigger check.
- **Read-modify-write** (load → change → save): two loaders overwrite each other. Fix: optimistic locking (version column, retry on conflict) for low contention; atomic in-place update (`UPDATE … SET n = n + 1`) when possible; `SELECT … FOR UPDATE` for genuine serialization.
- **The multi-instance trap**: an in-process mutex/lock protects NOTHING once you run 2 replicas (which Kubernetes will). Cross-instance coordination lives in the shared store: DB constraints, advisory locks, Redis locks with TTL + fencing awareness — or redesign so coordination isn't needed (partition by key).

## Locks, when you must

Smallest possible scope · consistent acquisition order everywhere (or deadlock) · never held across I/O or network calls · always released on the error path (finally/defer/use). Two locks needed regularly = the design wants a queue or a single owner instead.

## Async/await & coroutines

- **Don't block the async runtime**: no synchronous I/O or `Thread.sleep`/CPU-heavy loops on the event loop or default coroutine dispatchers — one blocked worker stalls unrelated requests. Blocking work goes to a dedicated pool (`Dispatchers.IO`, worker threads).
- **Structured concurrency**: children live inside a scope that awaits or cancels them. Fire-and-forget tasks (`GlobalScope.launch`, unawaited promises) = silent failures and leaks; every task has an owner who sees its error.
- **Cancellation is part of the contract**: honor it (check/propagate), clean up on it, and never swallow the cancellation exception in a broad catch.
- An unhandled rejection/exception handler at the runtime level is the safety net, not the strategy.

## Queues and background work

Bounded queues + defined backpressure (block, shed, or spill — choose consciously; unbounded = OOM later). Consumer concurrency breaks ordering — need order per entity → partition by key. Retries × concurrency ⇒ handlers idempotent (see error-resilience).

## JVM/Kotlin notes

Prefer high-level tools (coroutines, ExecutorService, CompletableFuture, java.util.concurrent collections/atomics) over raw `Thread`/`synchronized`. Visibility matters: unsynchronized non-volatile fields read by other threads may show stale values — use atomics/volatile/proper handoff. ThreadLocal + pooled threads/coroutines = leaked context bugs.

## Red flags — thoughts that mean STOP

| Thought | Reality |
|---|---|
| "This race is astronomically unlikely" | At 1k rps, one-in-a-million happens hourly. Design it out. |
| "It passes my tests" | A single-threaded test proves scheduling luck, not safety. Stress it. |
| "I'll add a mutex" | An in-process lock protects nothing at 2 replicas. Coordinate at the shared store. |
| "A more careful check before writing fixes it" | The check-then-act gap remains, just narrower. Atomic op or constraint. |
| "Fire-and-forget is fine here" | An unowned task fails silently and leaks. Give it a scope and an error owner. |
| "A short sleep fixes the flakiness" | Sleep = the same race with a timer attached. Await the actual condition. |

## Verification

Concurrency code gets a stress test (N workers hammering the invariant, assert it held) — a single-threaded unit test proves nothing. "Couldn't reproduce" means "didn't load it", not "fixed". If the invariant lives in the DB (unique, atomic update), the test is cheap and honest: fire concurrent requests, count the survivors.
