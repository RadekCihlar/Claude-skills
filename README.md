# Claude-skills

Global Claude Code configuration: one `CLAUDE.md` of always-on rules plus a suite of lazy-loaded skills. Goal: make any model in Claude Code — Haiku, Sonnet, Opus — work with the precision, verification discipline, and honesty of the strongest tier.

## Layout

- **`CLAUDE.md`** — the always-on core, loaded into every session. Hard rules (secrets, git, irreversible ops), working discipline (smallest correct change, read-before-edit, finish-the-turn), verification and honest-status rules, communication contract, skill/agent policy. Kept deliberately small: every token here is paid in every session.
- **`skills/`** — domain playbooks loaded only when the task matches their trigger. Each is a dense checklist, not an essay.

## Skills

- **frontend-standards** — anti-AI-slop catalog, positive design defaults, stack fallbacks, mandatory pre-done AI-tell scan.
- **security-perf-preflight** — trust-boundary security checklist (injection, traversal, SSRF, authz, secrets) + performance budget, applied before writing code.
- **repo-recon** — earning the right to edit an unfamiliar codebase: parallel search, convention capture, API-truth verification against installed versions.
- **api-design** — endpoints, error shape, cursor pagination, idempotency, versioning, compatibility rules.
- **db-and-migrations** — constraints in the DB, index discipline, N+1, zero-downtime expand-contract migrations.
- **testing-strategy** — what to test at which level, test construction, flaky-test triage, coverage as flashlight not target.
- **refactoring-safely** — behavior-preserving steps, characterization tests, refactor/behavior commit separation, strangler pattern.
- **error-resilience** — timeouts everywhere, retry policy (idempotent + backoff + jitter), degrade vs fail-fast, error messages for the 3am reader.
- **observability** — structured logging, level discipline, no secrets/PII, correlation IDs, metric cardinality, actionable alerts.
- **seo-technical** — canonicals, head hygiene, structured data, crawlability, reciprocal hreflang, Core Web Vitals budgets.
- **git-surgery** — reflog recovery, safe-undo decision table, automated bisect, conflict archaeology.

Process-level skills (TDD loop, systematic debugging, brainstorming, verification-before-completion) come from the [superpowers](https://github.com/obra/superpowers) plugin; communication and minimalism modes from the caveman and ponytail plugins. This repo's skills are the domain layer on top and assume nothing if those are absent.

## Install

```bash
git clone https://github.com/RadekCihlar/Claude-skills.git
# copy CLAUDE.md and skills/ into ~/.claude/  (or clone into an empty ~/.claude and let
# Claude Code create its runtime state around it — the .gitignore tracks only these files)
```

Skills register at session start; no further wiring needed.

## Design notes

- Rules are written for the weakest reader: short imperatives, one clause per rule, hard NEVERs isolated at the top, five memorizable prime directives.
- Anything long-form lives in a skill so it costs tokens only when triggered.
- The `.gitignore` is an allowlist — everything in `~/.claude` (settings, credentials, projects, history) stays untracked by default; only `CLAUDE.md`, `README.md`, and `skills/` are versioned.
