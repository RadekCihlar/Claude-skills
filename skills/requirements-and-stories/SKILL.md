---
name: requirements-and-stories
description: Use when asked to write or refine user stories, acceptance criteria, requirements, or tickets — break down an epic or feature, groom/refine a backlog item, define scope, or estimate work.
---

# Requirements & Stories

A story is a testable slice of user value. If you can't write its acceptance test, it isn't ready to build — the gap you can't test is the gap that ships broken.

## Story form

- Plain "user can X" beats ceremonial As-a/I-want/so-that unless the persona actually changes the behavior (admin vs visitor). Never costume internal work as a fake user story ("as a developer I want a refactor") — call it a task, that's fine.
- **Vertical slices** — thinnest end-to-end path (UI to DB) that a user can touch. Horizontal layer stories ("DB story", "API story", "UI story") ship nothing until all land and hide integration risk until the worst moment.
- Small: fits in a few days. Bigger → split by: happy path first / one persona / one platform / manual-before-automated fallback.

## Acceptance criteria

- Given/When/Then or a checklist — every criterion **observable and testable** by someone who didn't write the code.
- **Sad paths are mandatory**: invalid input, permission denied, empty state, upstream down, double-submit. Happy-path-only AC is half a story and the missing half is where the bugs are.
- Non-functional needs that matter get a criterion with a number: "list loads < 1s at 10k rows", "form usable by keyboard". Unstated NFRs don't exist.
- AC map 1:1 to tests later → `testing-strategy`.

## Breaking down an epic

1. **Walking skeleton first** — the thinnest end-to-end slice that proves the architecture (may return hardcoded data). Everything after is an increment on a working system.
2. Each increment independently shippable and demoable; ordering by risk (unknowns first), then value.
3. Unknowns become **spikes**: timeboxed, with a question to answer and a deliverable (a decision, a prototype conclusion) — "research X" without a question is a vacation.
4. Feature needing real design before slicing → `design-docs` (forward mode) / `architecture-decisions` for the expensive-to-reverse calls.

## Estimation

- Estimate **relative to a reference story** everyone knows, not in absolute hours pulled from air.
- Uncertainty is expressed as a range or a split, never false precision ("3–8 days, depends on whether the API supports batch" beats "5 days").
- Anything estimated over ~a sprint → split before estimating; large estimates are guesses with confidence intervals wider than the number.
- **Record the assumptions on the ticket** ("assumes existing auth covers this", "assumes no migration needed") — an estimate whose assumptions die silently becomes a lie retroactively.

## Definition of ready / done — keep them short

Ready: AC written incl. sad paths · dependencies named · testable. Done: AC pass · tests exist · deployed/demoable · docs touched if behavior changed. Longer lists get ignored.

## Checklist

Value visible per story · vertical slice, not layer · AC testable + sad paths present · NFRs have numbers · epic starts with walking skeleton · spikes timeboxed with a question · estimates relative, ranged, assumptions recorded.
