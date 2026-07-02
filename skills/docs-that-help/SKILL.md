---
name: docs-that-help
description: Use when writing or restructuring READMEs, runbooks, onboarding docs, changelogs, or architecture docs — documentation aimed at a reader with a job to do right now.
---

# Docs That Help

Every doc has ONE reader with ONE job. Write for that moment; delete everything else. Wrong or stale docs are worse than none — they burn trust and time.

## The four docs and their readers

- **README — the newcomer, 10 minutes to running.** Prereqs with versions → the 2-3 commands to run it → where things live (one-paragraph map) → where to go next. Copy-paste-ready commands, verified by actually running them. Not a features brochure.
- **Runbook — the 3am operator.** Symptom-indexed, imperative: *symptom → what to check (exact command/dashboard link) → action → how to verify recovery → when to escalate.* No theory, no history. If an alert can fire, its runbook line exists (see observability); if you handled an incident manually, the commands you ran become the next runbook entry.
- **ADR — the future maintainer asking "why is it like this?"** Context, decision, alternatives-with-reasons, accepted consequences. One page. (Details in architecture-decisions.)
- **Changelog — the upgrader.** Breaking changes first with migration steps, then features, then fixes. Written from the user's perspective ("`config.port` renamed to `server.port`"), not commit archaeology.

## Principles

- **Document the WHY and the non-obvious.** Code already documents the what; comments/docs restating it rot on the first refactor. Setup quirks, invariants, "this looks wrong but is load-bearing because…" — that's the payload.
- **Runnable beats prose.** A `make dev` / script with a one-line description beats four paragraphs of steps. If a procedure is executed more than twice, script it and document the script.
- **The two-question rule**: answered the same "how do I…?" twice → it goes in the doc now, while the answer is fresh.
- **Docs live next to code and ride in PRs** — same review, same versioning, visible in diffs. The wiki no PR touches is where information goes to die.
- **One entry point.** The README links to everything else; five competing entry docs means none is trusted. On finding stale docs: fix or delete on the spot — flagging them "may be outdated" helps nobody.
- Diagrams: ONE system-context diagram beats ten detailed ones that rot; keep them as code (Mermaid) so they diff and live in the repo.
- Write plainly: short sentences, concrete nouns, no internal codenames without a glossary line. A newcomer reading it has no shared context — that's the point of writing it down.

## Checklist

Reader + their job named · commands copy-paste-verified · why over what · runbook symptom-indexed with verification step · breaking changes lead the changelog · doc lives in-repo, reviewed · stale content deleted not hedged.
