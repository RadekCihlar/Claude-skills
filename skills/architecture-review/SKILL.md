---
name: architecture-review
description: Use AFTER a feature or system is implemented, when asked to verify the architecture, check that the implementation matches the design doc/HLD/LLD, find architecture drift, or run a post-build/post-merge design conformance review — reconciles design doc vs actual code vs decisions recorded in ADRs and memory.
---

# Architecture Review (conformance)

Three sources of truth exist after a build: the **design doc** (intent), the **code** (reality), and **ADRs + memory entries** (decisions made mid-flight that changed the intent). A review reconciles all three — every mismatch gets a verdict and an action, executed now while context is hot.

## 1. Gather

- The design doc / HLD / LLD for the work (none exists → that IS the first finding; write one via `design-docs`, reverse mode).
- ADRs touching the area.
- Project memory: read the MEMORY.md index, open every entry touching this feature — mid-build decisions and gotchas live there, not in the doc.
- The actual shipped change: the diff/PR range or the current state of the named modules.

## 2. Walk the claims

For EVERY claim in the design doc — each boundary, interface, flow, constraint, diagram arrow:

- Grep the code that implements it; read the real signature/route/call chain.
- Diagrams are claims too: each arrow must correspond to a real call you found.
- Record a verdict per claim. No claim gets skipped because it "obviously" holds.

## 3. Verdicts

| Verdict | Meaning | Action |
|---|---|---|
| **Conforms** | code matches doc | note it, move on |
| **Doc-drift** | code is right, doc is stale | fix the doc NOW, in this pass |
| **Violation** | code diverged with no recorded decision | fix the code, or ADR the divergence if it's actually better |
| **Undocumented** | code has structure no doc/decision records | add to doc — or question whether it should exist at all |

Deciding doc-drift vs violation = deciding which side is truth. Deliberate change someone chose → doc-drift (record why). Accidental divergence → violation.

## 4. Memory reconciliation

- Every mid-flight decision recorded in memory → must now appear in the design doc/ADR, or be consciously superseded (then update or delete the memory entry — stale memory misleads the next session).
- Every recorded gotcha → verify it is still true in the shipped code; fix the entry if not.
- Decisions made during THIS review (accepted divergences, new constraints) → write them to memory + doc before closing.

## 5. Output

A conformance table — claim → verdict → action taken — plus the doc/memory edits already applied. Findings-only prose ("mostly matches, some drift") is a failed review: every drift row shows the fix.

## Red flags

| Thought | Reality |
|---|---|
| "It basically matches" | Untested claim. Grep it. |
| "Doc update can wait for a follow-up" | It won't happen. Fix while context is hot — that's the point of reviewing now. |
| "Small drift, not worth recording" | Drift compounds silently. Record or revert, nothing in between. |
| "Memory files aren't part of architecture" | They hold the decisions the doc missed. Reconcile them or the next session re-litigates. |

## Handoffs

Doc fixes → `design-docs`. Divergence worth keeping → `architecture-decisions` (ADR it). Code must move back → `refactoring-safely`. Dead scaffolding found → `code-hygiene`.

## Checklist

Doc + ADRs + memory entries + real diff all gathered · every claim (incl. diagram arrows) grep-verified · verdict per claim, none skipped · doc fixed in-pass, not deferred · memory entries updated/deleted to match reality · new decisions written to memory + ADR · output is a claim→verdict→action table.
