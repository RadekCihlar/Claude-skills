---
name: code-hygiene
description: Use when asked to clean up code, remove unnecessary or noisy comments, delete dead code, fix naming drift, or tidy a module/file — what to delete vs keep, dead-code verification, and a behavior-preserving sweep discipline.
---

# Code Hygiene

A cleanup sweep is a refactor with zero behavior change and maximum deletion. The output is a smaller, quieter diff — never a "better" one. New abstractions, restructuring, or behavior fixes found along the way get flagged, not done (see refactoring-safely for restructuring).

## Comment taxonomy

**Delete on sight:**
- Narration — restates the next line (`// increment counter`, `// loop over users`).
- Changelog/attribution comments (`// modified by JK 2019-03`, `// fixed bug #123`) — git blame does this job with authority.
- Commented-out code — git history keeps it forever; the file shouldn't. No "might need later".
- Section banners (`//======= HELPERS =======`) when the language has real structure for it.
- Redundant doc-comments that add no information (`@param name the name`).
- Stale TODOs with no owner, ticket, or date — either file the ticket and link it, or delete.
- Auto-generated noise the generator will re-add.

**Keep (and never delete):**
- WHY comments — rationale the code can't show ("retry twice because upstream flaps on deploy").
- Invariants and warnings ("must run before X", "this looks wrong but is load-bearing because…", `ponytail:` ceiling markers).
- Links to issues, specs, RFCs, incident postmortems.
- Legal/license headers; doc-comments that are the project's convention for public APIs.
- Suppression comments WITH their justification (`eslint-disable … because Z`).

Judgment call: a comment explaining confusing code → first ask if renaming/extracting makes the code self-explanatory (that's the better fix, but it's a refactor — flag if beyond scope).

## Dead code — verify before deleting

"Looks unused" is not evidence. Before removing anything: grep callers AND string references (reflection, DI containers, route tables, config files, templates, tests, serialization all reference by NAME). Exported/public symbols may have external consumers — library code needs a deprecation cycle, not a delete. Feature-flagged branches: dead only if the flag is retired. When verified dead: delete completely — the function, its imports, its tests, its types — half-deleted code is new lint.

## Other noise in a sweep

Unused imports/variables · leftover debug output (console.log, print, dumps) · duplicated constants drifting apart (point at one) · trivially inconsistent naming within the touched file (match the file's own convention — repo-wide renames are a separate task) · files that ship nothing (empty modules, orphaned fixtures).

## Sweep discipline

- **Tests green before AND after; zero behavior change.** No test coverage on the target → smoke-check what you can, say so in the handoff.
- **Batch by kind, not by whim**: one commit for comments, one for dead code, one for imports/debug — a reviewer approves each in seconds. Never mix hygiene into a feature/fix commit.
- **Scope = what was asked.** Sweeping a module ≠ license to roam the repo. Adjacent mess → one `⚠️ Tech debt:` line each.
- Formatting churn only via the project's formatter, in its own commit, if asked.

## Red flags — thoughts that mean STOP

| Thought | Reality |
|---|---|
| "This comment might help someone someday" | If it restates the code, it helps nobody and rots. Delete. |
| "I'll keep the commented-out block just in case" | Git is the just-in-case. Delete. |
| "Nothing references it, quick delete" | Reflection/DI/routes/configs reference by string. Grep names first. |
| "While cleaning, I'll restructure this too" | That's a refactor with different rules. Flag it, stay in the sweep. |
| "It's only cleanup, tests can wait" | Zero-behavior-change is a claim. Green before and after is the proof. |
| "I'll also reformat everything" | Formatting churn buries the real diff. Own commit, formatter only, if asked. |

## Checklist

Comments: narration/changelog/commented-out deleted, WHY/invariants kept · dead code grep-verified incl. string refs, deleted completely · debug output + unused imports gone · commits batched by kind · tests green before/after · scope respected, findings beyond it flagged not fixed.
