# Claude-skills

A portable "work ethic" for [Claude Code](https://claude.com/claude-code): one global `CLAUDE.md` of always-on rules plus 32 lazy-loaded skills covering the full development lifecycle — architecture, backend, data, frontend, containers, Kubernetes, CI/CD, and production operations. The goal is that any model in Claude Code — Haiku, Sonnet, Opus — works with the precision, verification discipline, and honesty you'd expect from the strongest tier, on any project, in any language.

## What it does

Claude Code reads `~/.claude/CLAUDE.md` at the start of every session and treats it as standing instructions. Skills placed under `~/.claude/skills/` register automatically and load only when a task matches their description. This repo versions both:

- **`CLAUDE.md`** — the always-on core. It's deliberately small (every line here costs tokens in every session) and is written for the weakest reader: short imperative rules, hard NEVERs isolated at the top, five memorizable prime directives, and a trigger table that routes tasks to the right skill.
- **`skills/`** — dense domain playbooks (60–90 lines each) that load on demand. A schema change pulls in the migration-safety checklist; touching an HTTP handler pulls in the trust-boundary checklist; the rest of the time they cost nothing.

## How it helps

Without standing rules, model behavior drifts with the model and the day: silent guesses instead of questions, "should work now" instead of verified output, drive-by refactors, hallucinated APIs, `.env` files read into context. The core rules pin down the failure modes that matter most:

- **Safety rails.** Never read or commit secret-bearing files; never commit/push unprompted; irreversible operations need fresh confirmation every time; never game a failing test to get green.
- **Honest status.** "Done/fixed/passing" only after running the check and seeing real output. Partial work is reported as partial, with what's left.
- **Surgical changes.** Smallest correct diff, match the codebase's existing patterns, no speculative abstraction, dependencies as a last resort.
- **Real answers.** Verify APIs and flags in source before using them; ask when two good approaches exist; finish the turn instead of ending on a plan.

The skills then raise the floor per domain — so a quick "add an endpoint" still gets pagination, an error shape, and idempotency thinking, and a "quick migration" still ships expand-contract safe.

## The skills

| Skill | Loads when you… | Carries |
|---|---|---|
| `frontend-standards` | build or restyle UI | anti-AI-slop catalog, design defaults, pre-done scan |
| `security-perf-preflight` | touch a trust boundary or hot path | injection/SSRF/authz checklist + perf budget |
| `repo-recon` | enter unfamiliar code | exploration sequence, API-truth verification |
| `api-design` | design endpoints/webhooks/interfaces | error shape, cursor pagination, idempotency, versioning |
| `db-and-migrations` | write schema, queries, migrations | constraints-in-DB, indexing, zero-downtime expand-contract |
| `testing-strategy` | decide what/how to test | test levels, construction, flaky-test triage |
| `refactoring-safely` | restructure working code | characterization tests, green-step discipline, strangler |
| `error-resilience` | call external services / handle I/O | timeouts, retry policy, degrade-vs-fail, error messages |
| `observability` | add logging/metrics/alerts | structured logs, levels, cardinality, actionable alerts |
| `seo-technical` | build public pages | canonicals, structured data, hreflang, CWV budgets |
| `git-surgery` | recover work / hunt regressions | reflog, safe-undo table, automated bisect |
| `docker-images` | write Dockerfiles/compose | multi-stage, caching, non-root, signals, no baked secrets |
| `kubernetes-deploys` | write manifests / debug pods | probes, resources, PDBs, graceful shutdown, triage tree |
| `cicd-releases` | design pipelines/releases | build-once-promote, migration gating, canary, flags, rollback |
| `incident-response` | prod is broken now | mitigate-first, what-changed triage, blameless postmortem |
| `architecture-decisions` | design systems/boundaries | modular-monolith-first, sync-vs-async, build-vs-buy, ADRs |
| `concurrency-async` | share state / go parallel | race patterns + fixes, multi-instance traps, structured concurrency |
| `auth-and-identity` | build login/permissions | sessions vs JWT, OAuth+PKCE, CSRF, IDOR, password storage |
| `caching-strategy` | add any cache | key variance, invalidation layers, stampede defense |
| `performance-profiling` | diagnose slowness | measure-first, flamegraphs, EXPLAIN, load testing |
| `http-networking` | fight CORS/proxies/TLS | XFF trust, timeout chains, cert expiry, connection pools |
| `dependency-upgrades` | bump deps / triage CVEs | cadence, major isolation, reachability triage, lockfile discipline |
| `docs-that-help` | write READMEs/runbooks | reader-with-a-job docs, symptom-indexed runbooks, ADRs |
| `skill-authoring` | grow this repo | house format, trigger-description craft, headless invocation probes |
| `code-hygiene` | clean up comments/dead code | comment taxonomy, dead-code verification, sweep discipline |
| `design-docs` | write HLD/LLD or document a system | forward + reverse design docs, grep-verified claims, Mermaid flows |
| `architecture-review` | verify built code matches the design | doc-vs-code-vs-memory reconciliation, verdict table, drift fixes |
| `reviewing-code` | review someone else's PR/diff | read-beyond-diff, verified findings, severity ladder, verdict-first |
| `requirements-and-stories` | write stories/ACs or split an epic | vertical slices, sad-path ACs, walking skeleton, honest estimates |
| `threat-modeling` | threat model a design | trust-boundary map, STRIDE, abuse cases, mitigations-with-owners |
| `infra-as-code` | write Terraform/Pulumi/Ansible | state discipline, plan review, drift reconciliation, secrets-in-state |
| `data-pipelines` | build ETL/batch/backfill jobs | idempotent re-runs, watermarks, quality gates, missed-run alerts |

Process-level behavior (TDD loop, systematic debugging, brainstorming, verification-before-completion) comes from the excellent [superpowers](https://github.com/obra/superpowers) plugin, and the terse-communication + minimal-build mode referenced in `CLAUDE.md` (RDX ultra) comes from the [rdxmin](https://github.com/JayPokale/RDXmin) plugin — one ruleset that unifies zero-fluff prose with YAGNI-first code, plus a deterministic tool-output compressor. Everything in this repo works standalone if you skip it — the skill trigger table simply routes to what's installed.

## Setup

`~/.claude` is Claude Code's config directory on every OS (`C:\Users\<you>\.claude` on Windows, `~/.claude` on macOS/Linux).

**Fresh machine (no `~/.claude` yet, or nothing in it you care about):**

```bash
git clone https://github.com/RadekCihlar/Claude-skills.git ~/.claude
```

Claude Code creates its runtime state (settings, history, projects) around the tracked files. The repo's `.gitignore` is an **allowlist** — only `CLAUDE.md`, `README.md`, and `skills/` are versioned, so settings and credentials can never be committed, even with `git add -A`.

**Existing `~/.claude` you want to keep:**

```bash
git clone https://github.com/RadekCihlar/Claude-skills.git /tmp/claude-skills
cp /tmp/claude-skills/CLAUDE.md ~/.claude/CLAUDE.md        # back up your own first
cp -r /tmp/claude-skills/skills ~/.claude/
```

Or turn your live `~/.claude` into the repo itself (what this setup does):

```bash
cd ~/.claude
git init -b main
git remote add origin https://github.com/RadekCihlar/Claude-skills.git
git fetch origin && git checkout -f -t origin/main   # overwrites CLAUDE.md/skills only
```

**Verify:** start a new Claude Code session and ask "which skills are available?" — the 32 above should be listed. Skills register at session start, so restart after changes (or run `/reload-skills` on Claude Code ≥ 2.1.152).

**Optional companions:** install the [superpowers](https://github.com/obra/superpowers) plugin for the process skills the trigger table references. Without it those rows are inert; nothing breaks.

## settings.json extras (copy-paste)

`~/.claude/settings.json` is deliberately **not** tracked (it holds machine/account state), so these two pieces need a one-time paste per machine. Merge them into your existing file — don't replace it; if you have no `hooks` or `env` key yet, paste the blocks as-is inside the top-level `{ }`.

**Memory enforcement after compaction.** When a long session compacts its context, this hook injects an instruction telling the model to persist durable decisions/gotchas/preferences to memory files before resuming — mechanical enforcement of CLAUDE.md rule 39, not model goodwill. (PreCompact can't do this: its output only reaches the debug log. `SessionStart` + `"compact"` matcher is the event whose stdout reaches the model.)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Context was just compacted. Before continuing the task: persist durable facts from this session not yet in memory - decisions code/git will not record, discovered gotchas, user corrections and preferences - one fact per file in the auto-memory directory plus a MEMORY.md index line (global CLAUDE.md rule 39). Then resume.'",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

**Always-on efficiency mode** (only if you use the [rdxmin](https://github.com/JayPokale/RDXmin) plugin referenced in CLAUDE.md's Always-On Modes section — harmless otherwise):

```json
{
  "env": {
    "RDX_DEFAULT_MODE": "ultra"
  }
}
```

Verify after pasting: the file must still parse (`python -c "import json;json.load(open('settings.json'))"` or any JSON validator) — a malformed settings.json silently disables everything in it.

## Customizing

- Edit `CLAUDE.md` for behavior you want in every session; keep it short — long files get skimmed by small models and every token is paid on every request.
- Add a skill by creating `skills/<kebab-name>/SKILL.md` with `name:` (must match the directory) and a trigger-rich `description:` in the frontmatter — the description is what makes it load, write it as "Use when…". The `skill-authoring` skill carries the full format and the headless verification protocol.
- Commit and push from `~/.claude` like any repo; the allowlist keeps it safe.

## Design principles

1. **Always-on core stays small; depth lives in skills.** Rules read on every request must earn their tokens; a 90-line migration playbook should cost nothing until you write a migration.
2. **Written for the weakest reader.** One clause per rule, concrete triggers, MUST/NEVER made explicit — big models tolerate ambiguity, small ones don't.
3. **Verifiable over aspirational.** Rules state observable behavior ("run the check, paste the output"), not virtues ("be careful").
4. **Nothing secret is versionable.** The allowlist `.gitignore` makes leaking credentials a two-mistake event instead of a one-mistake event.
