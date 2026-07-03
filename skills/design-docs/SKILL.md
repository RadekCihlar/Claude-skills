---
name: design-docs
description: Use when asked for an HLD, LLD, design doc, architecture diagram, sequence diagram, C4/Mermaid/UML diagram, high-level or low-level design, technical/solution design, architecture overview, or system documentation — and when asked to document, map, or reverse-engineer the architecture, data flow, or how an existing app/system/service works from its actual code. Also for design docs of new features before building.
---

# Design Docs (HLD / LLD)

A design doc is decisions + interfaces + flows that a reader can verify. Every claim about existing code must be grep-verified in source; everything not yet built is explicitly marked **proposed**. A design written from memory is fiction with diagrams.

## Pick the mode first

- **Forward** — designing something not yet built (feature, service, rewrite). Output guides implementation.
- **Reverse** — documenting what exists. Output = ground truth for the next maintainer; investigate before writing a single sentence.
- Small system → one doc with an HLD half and an LLD half. Don't produce two ceremonial documents where one page serves.

## Architect stance (applies to both modes)

- Constraints before structure: real load numbers, team size, deadline, existing stack. No pattern the constraints don't justify.
- Every boundary named with an owner and a reason; every external dependency listed with its failure behavior (what happens when it's down?).
- Significant decisions (expensive to reverse) get alternatives-with-reasons → that part is an ADR: ALSO load `architecture-decisions`.
- Failure modes are part of the design, not an appendix: at least one failure flow documented alongside the happy paths.

## HLD contents (context + containers)

1. Purpose in one paragraph — what the system does and for whom.
2. System-context diagram (Mermaid, in-repo): the system, its users, every external system. ONE diagram; ten detailed ones rot.
3. Deployables/containers and what runs where (runtime, deployment topology).
4. External dependencies table: name → protocol → failure behavior → owner.
5. Primary data flows (where data enters, transforms, rests, leaves).
6. Cross-cutting: authn/z model, observability, scaling limits, known ceilings.

## LLD contents (components + interfaces)

1. Component map inside each container: module → responsibility → key entry points (file paths).
2. Interface specs — endpoints, events, public functions: ALSO load `api-design`.
3. Data model / schema with ownership: ALSO load `db-and-migrations`.
4. Sequence diagrams (Mermaid) for the 2-3 core flows plus 1 failure flow — every arrow is a real call (reverse mode) or a designed one (forward, marked proposed).
5. Error handling and config/env surface.

## Reverse mode — investigation before prose

- Run the `repo-recon` sequence first, but doc-depth: entry points → routing/wiring → follow the real call chains of the core flows.
- Build diagrams from actual imports, routes, and callers you found — never from what the architecture "should" be. Can't verify a claim → write "unverified", don't invent.
- Recover the WHY from git log, ADRs, code comments, and the project's memory files — the doc must explain load-bearing oddities, not just structure.
- Found an existing stale design doc → replace or fix it in place; never append a competing second doc.

## Craft (from docs-that-help)

Reader + their job named at the top · Mermaid diagrams live in the repo and ride PRs · why over what · plain words, no internal codenames without a gloss · runnable references (file paths, commands) over abstract description.

## Handoffs

Decisions inside the doc → `architecture-decisions` (ADR). Interfaces → `api-design`. Schema → `db-and-migrations`. **After the design is implemented → `architecture-review`** to verify the built system still matches this doc.

## Checklist

Mode chosen (forward/reverse) · constraints stated with numbers · context diagram + deps-with-failure-behavior present · core flows + 1 failure flow as sequence diagrams · every existing-code claim grep-verified, proposals marked proposed · WHY captured, not just structure · decisions ADR'd · doc lives in-repo · stale predecessors deleted.
