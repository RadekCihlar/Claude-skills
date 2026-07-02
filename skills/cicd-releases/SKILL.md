---
name: cicd-releases
description: Use when designing or editing CI/CD pipelines, release/rollback strategy, canary or blue-green deploys, feature flags, or deciding how code+migrations reach production.
---

# CI/CD & Releases

The pipeline's job: make deploying boring. Small, frequent, reversible releases beat big careful ones — risk scales with batch size, not with deploy count.

## Pipeline shape

- **Build once, promote everywhere.** One immutable artifact (image/jar) built after tests pass, deployed unchanged through staging → prod. Rebuilding per environment = testing one thing, shipping another.
- Stage order: lint + unit (fast, fail early) → build artifact → integration tests against the artifact → deploy staging → smoke → deploy prod. Anything slower than ~10 min total gets parallelized or trimmed — slow pipelines rot into "skip CI" culture.
- **Migrations run as a pipeline step BEFORE the new code rolls out**, and must be backward-compatible with the still-running version (expand-contract — see db-and-migrations). Migration failure aborts the deploy, leaving old code on old-compatible schema.
- CI secrets: short-lived OIDC federation to cloud providers over long-lived stored keys; least-privilege per job; masked in logs.

## Rollback — a first-class feature, not an emergency hack

- Previous N artifacts stay deployable with one command; practice the rollback path before you need it.
- **Roll back code, never migrations** — schema fixes go forward. This only works if migrations were expand-contract; that discipline is what buys push-button rollback.
- Define the auto-rollback trigger up front: error rate / p99 over threshold for M minutes after deploy → revert without a meeting.

## Progressive delivery

- **Canary**: route a small % (or one replica) to the new version, gate promotion on real metrics (errors, latency, business KPI) — a canary nobody watches is just a slow deploy.
- **Blue-green** when instant cutover/rollback matters and state allows two parallel stacks.
- **Feature flags decouple deploy from release**: ship dark, enable per cohort, kill-switch risky paths. Flags are debt with an expiry — dead flags get deleted in the sprint after full rollout, or the codebase becomes an if-forest.

## Release hygiene

- Deploy when people can respond — not at the end of the last person's day.
- Every deploy verified: automated smoke + one dashboard cycle (error rate, latency, saturation) before walking away. "Pipeline green" ≠ "prod healthy".
- Version discipline: semver for libraries (breaking = major, no exceptions); services = build metadata + changelog generated from commits.
- The pipeline is code: reviewed, versioned, reproducible locally where possible. A deploy step only a human can do is an incident scheduled for later.

## Checklist

One artifact promoted, never rebuilt · migrations gated + backward-compatible · rollback rehearsed, trigger defined · canary gated on metrics · flags have owners + expiry · CI secrets short-lived + least-privilege · post-deploy verification automatic.
