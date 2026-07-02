---
name: skill-authoring
description: Use when creating a new skill, editing an existing SKILL.md, or a skill fails to trigger/register — covers the house format, trigger-description craft, and the headless verification protocol that proves a skill actually auto-invokes.
---

# Skill Authoring

A skill is knowledge that loads itself at the right moment. Two failure modes: it never triggers (bad description), or it triggers and doesn't help (essay instead of checklist). Both are testable — never ship a skill on hope.

## Should this be a skill at all?

- Recurring domain knowledge that a model gets wrong without guidance → skill.
- One-project facts → project CLAUDE.md or memory. Always-on behavior rules → global CLAUDE.md (they can't rely on triggering). Already covered by an installed plugin (superpowers etc.) → don't duplicate; two skills competing for one trigger means the better one loads half the time.

## House format

- Location `~/.claude/skills/<kebab-name>/SKILL.md`; frontmatter `name:` MUST equal the directory name (mismatch = silently never loads).
- **Description is the product.** It's the ONLY text the model sees when deciding to load. Write it as a matching rule against what the user actually says — symptoms, artifacts, verbs: "Use when … CrashLoopBackOff, editing a Dockerfile, CORS errors", not "Kubernetes best practices". List concrete trigger nouns; weak models match words, not concepts.
- Body: one-line thesis at top, then dense imperative sections — checklists and decision tables, no essays. 50–90 lines; past ~120 it gets skimmed, and skimmed guidance is no guidance.
- End with a scannable **Checklist** gate.
- **Red-flag table** (`| Thought | Reality |`) only where the failure mode is *rationalization* ("small table, direct ALTER is fine"), not knowledge gaps — on every skill the device dilutes.
- **Cross-reference siblings explicitly** ("pod-level failure → ALSO load `kubernetes-deploys`"): weak models tend to load exactly one skill, so the loaded one must route onward.

## Registration mechanics

Skills scan at session start. Mid-session: user runs `/reload-skills` (the model cannot). New skill invisible → check name/dir match and frontmatter fences first.

## Verification protocol — before trusting any skill

1. **Lint**: first line `---`, `name:` matches dir, `description:` starts with a trigger phrase ("Use when/BEFORE…").
2. **Registration probe** (fresh session, no restart needed):
   `claude -p --model haiku "Which of these exact names appear in your available-skills list: <name>? Reply with the subset or NONE."`
3. **Auto-invocation probe** — the real test. Give the WEAKEST model a realistic user prompt that never mentions skills:
   `claude -p --model haiku --verbose --output-format stream-json "<realistic task, e.g. 'my pod is in CrashLoopBackOff after a deploy'>" > out.jsonl`
   then search `out.jsonl` for `Launching skill: <name>`. No hit → the description doesn't match how users phrase the problem; add their words to it and re-probe. Iterate description, not hope.
4. Wire it in: add a row to the global CLAUDE.md trigger table and the repo README table; commit skill + wiring together.

## Anti-patterns

Vague description ("helps with databases") · essay prose without checklists · duplicating an installed plugin's scope · >120 lines · advice that rots datefast without marking it (versions, thresholds — date or generalize) · shipping without the invocation probe.

## Checklist

Scope not already covered · name = dir · description written in user-words with concrete triggers · ≤~90 dense lines + checklist gate · siblings cross-referenced · registration + auto-invocation probes green on haiku · trigger table + README wired · committed together.
