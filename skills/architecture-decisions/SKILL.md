---
name: architecture-decisions
description: Use when designing a system or module structure, deciding boundaries, choosing sync vs async communication, splitting or merging services, build-vs-buy calls, or writing an ADR.
---

# Architecture Decisions

Architecture = the decisions that are expensive to reverse. Spend scrutiny proportional to reversibility: cheap-to-change gets a default and a note; expensive-to-change gets alternatives, an ADR, and a night's sleep.

## Start from constraints, not aesthetics

Actual load (numbers, not vibes), team size, ops maturity, deadline, existing stack. The right architecture for 3 devs and 200 rps is different from 40 devs and 20k rps — resume-driven design (microservices, event sourcing, k8s for a CRUD app) is the most common failure.

## Boundaries

- **Modular monolith first.** Modules with strict interfaces inside one deployable. A network boundary adds latency, partial failure, versioning, and distributed debugging — pay that tax only for a reason: independent scaling profile, independent team ownership/deploy cadence, hard failure isolation, or a different runtime need.
- Cut along **business capabilities** (billing, ingestion, notifications), not technical layers (a "services layer service"). High cohesion inside; narrow, explicit interface outside.
- **Each module owns its data.** Cross-module access goes through the interface, not through reaching into another module's tables — shared tables weld modules together permanently.
- A boundary you can't name a single owner and reason for is decoration.

## Communication

- **Sync (call) for queries and anything the caller must know succeeded now.** Keep call chains shallow — sync chains multiply latency and couple availability (A→B→C = A's uptime is the product).
- **Async (queue/event) for side effects across boundaries** — notify, index, analytics: things that can lag and retry. Requires the error-resilience discipline: idempotent consumers, DLQs, monitoring lag.
- Events-everywhere has a real price: debugging becomes archaeology. Default to boring calls; earn each event.

## Build vs buy

Differentiating core → build. Commodity (auth, payments, email, search, monitoring) → managed/off-the-shelf until scale or requirements genuinely force otherwise. The build option's true cost includes ops, security patches, and the person who leaves.

## State & data

The database choice and schema outlive everything — decide data ownership and consistency needs first (what must be transactional together = same store, same module). Distributed transactions are a smell that a boundary is in the wrong place. Cross-service consistency → sagas/outbox, accepted eventual-ness, or redraw the boundary.

## Record it — ADR

One page per significant decision, committed next to the code: **Context** (forces, constraints) → **Decision** → **Alternatives considered** (with the real reasons they lost) → **Consequences** (including the bad ones you're accepting). Write it when deciding, not retroactively. The alternatives section is the valuable part — it stops the same debate from re-running every year.

## Checklist

Constraints stated with numbers · boundary has owner + reason · data ownership decided · sync/async chosen per interaction, not ideology · build-vs-buy includes ops cost · irreversible parts got alternatives + ADR · no pattern present that the load/team don't justify.
