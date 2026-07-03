---
name: reviewing-code
description: Use when asked to review a commit, PR, MR, diff, branch, or any code changes — "review this", "code review", "check this PR", "look over these changes", a second opinion before merge. Structured review with verified findings, a severity ladder, and a verdict, not a style-nit flood.
---

# Reviewing Code

A review exists to catch what breaks production and what misleads the next maintainer — in that order. "LGTM" without reading and a flood of style nits both fail the author equally.

## Ground rules

- **Read beyond the diff.** Bugs live at the boundary of the change: read the callers of changed functions, the definition of types the diff touches, the tests that cover the area. A diff-only review verifies syntax, not behavior.
- **Verify every claim before writing it.** "This might break X" → grep X and check. A finding you didn't verify is a guess wearing a review's authority. Can't verify → say "unverified, please confirm" explicitly.
- Run what you can: build, typecheck, the co-located tests. A red check found by the reviewer beats one found in CI an hour later.

## Review order

1. **Correctness vs stated intent** — does the change do what the PR description says? Walk the edge cases: empty/null input, error paths, concurrent calls, large inputs, timezone/encoding.
2. **Trust boundaries** — anything touching user input, queries, paths, redirects, authz → run the `security-perf-preflight` checklist against the diff.
3. **Tests** — new behavior has a test; the failure path has a test; tests assert outcomes, not implementation. Missing harness ≠ excuse for untested logic (ask for a smoke check). Test smells → `testing-strategy`.
4. **Blast radius** — API/contract compatibility (`api-design`), migration safety in the diff (`db-and-migrations`), rollback story, feature-flag guarding for risky paths.
5. **Maintainability last** — naming, dead code, scope creep ("while I was here" changes belong in their own PR).

## Severity ladder — label every finding

- **Blocker** — correctness, data loss, security, broken migration. Must state the concrete failure scenario: inputs/state → wrong outcome.
- **Should-fix** — bug-in-waiting: missing error path, unbounded growth, N+1 on a hot path, misleading name on a public surface.
- **Nit** — style/preference, prefixed `nit:`, never blocks a merge. More than ~5 nits → one pattern-level comment instead of a barrage.

## Output shape

Verdict first: **approve / approve-with-nits / request-changes**. Then findings ordered by severity, each with `file:line`, what breaks, and a concrete fix (or a diff suggestion). Close with what you did NOT review (didn't run it, skipped generated files) — honesty about coverage is part of the review.

## Red flags

| Thought | Reality |
|---|---|
| "LGTM, it's a small diff" | Small diffs delete backups. Read the callers. |
| "The author surely tested this" | The reviewer is the test the author can't skip. Check. |
| "I'll comment on style, that's something" | Style comments without a correctness pass = negative value: false confidence. |
| "This might be a problem" (unchecked) | Verify or mark unverified. Guesses erode trust in real findings. |

## Checklist

Read callers/tests, not just diff · every finding verified or marked unverified · trust-boundary diff got the preflight checklist · tests cover new behavior + failure path · severity labeled, blockers have failure scenarios · verdict stated first · unreviewed areas disclosed.
