---
name: infra-as-code
description: Use when writing, editing, or reviewing Terraform, OpenTofu, Pulumi, CloudFormation, or Ansible — infrastructure-as-code modules, state management, plan/apply workflow, drift, imports, or provisioning cloud resources by code.
---

# Infrastructure as Code

IaC is code whose bugs delete databases. The plan/diff is the review artifact; the state file is the crown jewels. Everything here serves two goals: no surprise destroys, no untracked reality.

## State discipline

- **Remote backend with locking, always** — even solo (local state + a laptop = single point of total loss).
- One state per environment / blast-radius unit. One giant shared state = one giant shared outage and eternal lock contention. Split by env first, then by lifecycle (network vs app).
- Never hand-edit state. Refactors use `state mv`, removals `state rm`, existing resources `import`. Renaming a resource in code without `moved`/`state mv` = destroy + recreate in the next plan.
- **State contains secrets in plaintext** (DB passwords, keys the provider returns). Access to state = access to secrets: lock the backend down and never commit state to git.

## Plan / apply workflow

- **Every apply is preceded by a read plan.** The words to hunt in plan output: `destroy`, `forces replacement`, `-/+`. A replacement of a stateful resource (DB, volume, bucket) needs explicit human confirmation — that's a data-loss event wearing a diff.
- Prod applies run from CI with an approval gate, not from laptops — laptop applies bypass review and use personal credentials. → `cicd-releases` for the gating.
- `-target` and `-refresh=false` are incident tools, not a workflow. Regular use means the state layout is wrong.

## Structure

- Environments = separate state + separate directory (or workspace) sharing versioned **modules**; differences expressed as variables, never copy-paste drift between env folders.
- Write a module on the rule of three (third repetition), not speculatively. Deeply nested modules are IaC's spaghetti — prefer flat composition.
- Pin provider and module versions; commit the lockfile; upgrades are deliberate PRs → `dependency-upgrades`.
- No secrets in `.tf`/`.yml` files or tfvars committed to git — pull at apply time from a secret manager / env / data source (and remember they still land in state, above).

## Drift

- Detect: scheduled `plan` in CI that alerts on non-empty diff.
- Reconcile: every manual console change gets imported into code or reverted — "I'll just fix it in the console" is how the next apply silently undoes an incident fix.

## Ansible-specific

Idempotency is the contract: modules over `shell:`; a `shell:` task needs `creates:`/`changed_when` so a re-run is safe. Check mode (`--check --diff`) is the plan equivalent.

## Red flags

| Thought | Reality |
|---|---|
| "Small change, skip reading the plan" | The plan is where `forces replacement` hides. Small diffs replace databases. |
| "I'll fix it in the console and codify later" | Later never comes; next apply reverts the fix mid-incident. |
| "One state file is simpler" | Until the lock, the 20-minute plans, and the blast radius of one bad apply. |

## Checklist

Remote locked backend · state split by env/lifecycle · plan read, replacements confirmed · prod applies via gated CI · versions pinned + lockfile committed · secrets via manager, state access locked · drift detection scheduled · module only on third repetition. Kubernetes manifests/Helm themselves → `kubernetes-deploys`; images → `docker-images`.
