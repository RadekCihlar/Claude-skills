---
name: dependency-upgrades
description: Use when bumping dependencies, responding to CVE/audit findings, planning a framework major-version migration, or untangling lockfile/peer-dependency conflicts.
---

# Dependency Upgrades

Small regular bumps are maintenance; a three-year jump is a project. Cadence is the whole game — the cost of upgrading grows superlinearly with the distance skipped.

## Routine bumps

- Patch/minor: batch them, run the full suite, skim changelogs only for anything security- or behavior-flagged. Semver is a promise, not a guarantee — the tests are the verdict.
- **Major: one dependency per commit/PR**, so a regression bisects to a single bump. Read the migration guide BEFORE bumping, not after the build breaks; run official codemods first, hand-fix the remainder.
- After any bump, exercise the feature that uses the dep, not just the unit suite — mocked boundaries hide integration breaks. Frontend: check bundle size delta too.
- Don't take day-0 majors into production-critical paths; let the ecosystem find the mines for a few weeks. (Pinning back a major that breaks peers — e.g. a linter whose plugins lag — is legitimate; record why and when to retry.)

## Security (CVE/audit) findings

- Prioritize by **exploitability in YOUR usage**, not raw count: a critical in a dev-only tool ≠ a moderate in your request-parsing path. Reachability matters — is the vulnerable function even called?
- Fix order: bump the direct dep → bump the parent that pins the vulnerable transitive → override/resolution as LAST resort with a comment naming the CVE and the exit condition (overrides silently pin forever and rot).
- Zero-day in a core lib: patch beats pretty — take the patch bump immediately, clean up later.

## Lockfiles & hygiene

- The lockfile is the truth and is always committed; CI installs frozen (`--frozen-lockfile` / `npm ci`) so builds are reproducible.
- Lockfile changes are intentional, reviewed, and never drive-by in unrelated PRs.
- Peer-dependency conflicts: understand WHO wants WHAT (`pnpm why`, `npm ls`) before forcing — `--force`/`--legacy-peer-deps` converts an error now into a runtime mystery later.
- Renovate/Dependabot: enable with grouping (all patches weekly) so the queue stays mergeable; an ignored bot is worse than no bot (100 stale PRs = alarm fatigue).

## Framework majors (the big ones)

Upgrade on a branch with green CI as the gate; incremental if the framework supports it (compat flags, codemods, running both module systems). Budget for the ecosystem: plugins/middleware lag the core — inventory which of YOUR deps depend on the framework before starting. Never combine a framework major with feature work in one change.

## Checklist

Majors isolated + guide read first · full suite + real-feature smoke after · CVEs triaged by reachability · overrides documented with exit condition · lockfile frozen in CI · bot PRs grouped and actually merged.
