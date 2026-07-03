---
name: threat-modeling
description: Use when asked to threat model a feature, system, or design, run a design-level security review or attack-surface assessment, or answer "how could this be abused / what could go wrong" BEFORE or without code. Complements security-perf-preflight, which is the code-level checklist.
---

# Threat Modeling

Four questions: what are we building, what can go wrong, what are we doing about it, did we actually do it. Answer them against the data flow, not against vibes — threats live where data crosses a trust boundary.

## 1. Map the system (what are we building)

Draw or describe the data-flow: **entry points** (endpoints, uploads, webhooks, message consumers, admin panels), **trust boundaries** (internet→app, app→DB, service→service, user→admin), **data stores** and what's sensitive in them, **external dependencies**. A reverse-engineered map must come from actual code/config → `design-docs` reverse mode. Every arrow that crosses a trust boundary gets analyzed; arrows inside one boundary mostly don't.

## 2. Enumerate threats (what can go wrong)

Per boundary-crossing element, walk **STRIDE**: Spoofing (fake identity), Tampering (modify data/requests), Repudiation (no trace of who did it), Information disclosure (data leaks — errors, timing, IDs), Denial of service (exhaust quota/CPU/storage), Elevation of privilege (user does admin things).

Then **abuse cases** — the feature working as designed, used hostilely: enumeration via sequential IDs, scraping, quota theft (your API keys burning for someone else's traffic), SSRF via user-supplied URLs, upload of active content, free-tier arbitrage, harassment vectors. Ask: what would a bot, a competitor, and a bored teenager each do with this?

## 3. Mitigate (what are we doing about it)

- Every threat gets: a **control** (authz check, rate limit, input validation, signed URLs, audit log, quota cap), an **accepted risk** (recorded, with owner and reason), or a **design change** (often cheapest — don't expose the ID at all).
- "Be careful" and "we'll monitor it" are not mitigations. A mitigation names the mechanism and where it lives.
- Rank what you fix first by exploitability × impact — internet-reachable, no-auth, cheap-to-automate goes to the top.

## 4. Verify (did we do it)

Controls become code-level checklist items → `security-perf-preflight` when writing, `reviewing-code` at review. Auth mechanism details → `auth-and-identity`. Design changes worth recording → `architecture-decisions` (ADR). After the build, the threat model's claims are review-checkable → `architecture-review`.

## Output shape

A table — element/flow → threat (STRIDE class) → mitigation or accepted-risk → status — plus a ranked top-N of risks worth acting on now. Small feature = ten rows, not a workshop.

## Red flags

| Thought | Reality |
|---|---|
| "Internal service — no threats" | SSRF, a leaked credential, or one phished laptop makes internal reachable. Boundaries, not network location. |
| "We validate on the client" | The client is the attacker's code. Server-side or it doesn't exist. |
| "Nobody would bother attacking this" | Bots bother. Scrapers and quota thieves target anything with an open port. |
| "We'll add security after launch" | Attack surface ships on day one; retrofits miss design-level flaws (IDs, flows). |

## Checklist

Data-flow mapped from real code/config · trust boundaries drawn · STRIDE walked per crossing · abuse cases (bot/competitor/teenager) considered · every threat → control, accepted risk w/ owner, or design change · ranked top-N stated · controls handed to code-level checklists.
