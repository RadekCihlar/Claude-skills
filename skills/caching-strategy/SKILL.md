---
name: caching-strategy
description: Use when adding or reviewing any cache — in-memory, Redis, HTTP/CDN, memoization — or debugging staleness, cache stampedes, and invalidation bugs.
---

# Caching Strategy

A cache trades correctness risk for speed. Take the trade only with a measured reason ("this query is 40% of p95"), because you're buying the two hard problems: invalidation and naming.

## Pick the highest level that works

Browser (`Cache-Control`/`ETag`) → CDN → app-shared (Redis/memcached) → app-local (per-instance memory) → DB (materialized views). Higher levels serve requests before they cost you anything. Don't hand-roll an app cache for something HTTP caching already solves.

## Keys — where the leaks live

The key must include EVERYTHING that varies the value: user/tenant for private data, locale, API version, feature-flag state if it changes the shape. The classic incident: per-user data cached on a shared key → users see each other's data. Private data on a shared cache needs the user ID in the key, or better, doesn't get cached. Prefix keys with a schema version — bumping the prefix is the cheapest "invalidate everything" there is.

## Invalidation — layered, never trusted

- **TTL always, as the backstop** — even "we invalidate on write" caches get a TTL, because your invalidation WILL have a bug and the TTL caps the damage.
- Correctness-sensitive → invalidate/update on write (write-through or delete-on-write). Delete-on-write beats update-on-write for simplicity (next read repopulates).
- Version-stamped keys sidestep invalidation entirely: content-hashed asset names, `user:{id}:v{n}`.
- Never cache: authorization decisions, anything where staleness = incident (balances, inventory at checkout), secrets.

## Stampede & hot keys

When a hot key expires, N concurrent requests all miss and hammer the origin at once. Defenses: **jittered TTLs** (± random %) so keys don't expire in herds; **single-flight** (one request repopulates, others wait or get stale); **stale-while-revalidate** (serve expired value, refresh in background). Any hot-key cache without one of these is a scheduled origin outage.

## Per-instance caches in replicated deployments

N replicas = N independent caches: inconsistent between pods, invalidation doesn't propagate, memory cost × N. Fine for immutable/slow-changing data (config with short TTL, static lookups). Wrong for sessions or anything mutated through one instance — that's what the shared tier is for. Bound every in-memory cache (max entries + eviction) — an unbounded cache is a memory leak with a nicer name.

## HTTP specifics

Immutable assets: hashed filename + `Cache-Control: public, max-age=31536000, immutable`. HTML/API: `no-cache` (revalidate) or short max-age + `ETag`/`Last-Modified` for cheap 304s. `Vary` on what actually varies the response (`Accept-Language`, NOT `User-Agent`). Authenticated responses: `private`, or CDNs will share them.

## Red flags — thoughts that mean STOP

| Thought | Reality |
|---|---|
| "We'll invalidate on every write, no TTL needed" | Your invalidation WILL have a bug. TTL backstop always. |
| "It's obviously slow, cache it" | No measurement = likely caching the wrong layer. Get the number first. |
| "One key works for everyone" | Locale/user/version variance means someone sees someone else's data. |
| "Just an in-memory map, keep it simple" | × N replicas = N inconsistent caches. Fine only for immutable data. |
| "Hot key expiring is no big deal" | N simultaneous misses = origin stampede. Jitter, single-flight, or SWR. |
| "We'll add a size bound later" | An unbounded cache is a memory leak with a nicer name. |

## Checklist

Measured reason to cache · highest workable level · key includes all variance (user/locale/version) · TTL backstop + chosen invalidation path · stampede defense on hot keys · in-memory caches bounded · nothing auth/critical-freshness cached · metrics: hit rate + origin load visible.
