---
name: git-surgery
description: Use when recovering lost work (reflog), hunting when a regression appeared (bisect), undoing mistakes safely (revert vs reset), or untangling messy history/conflicts. Safe-undo playbook — destructive operations always get explicit user confirmation first.
---

# Git Surgery

Git almost never loses committed work — it loses track of it. Find it, point a ref at it, done. Rule zero: before any history-touching operation, know your escape hatch (`git reflog` + the current SHA).

## Recovering "lost" work

- **`git reflog`** — every position HEAD has held. Find the SHA from before the mistake → `git branch rescue <sha>` (safe: creates a pointer, changes nothing) → inspect, then merge/cherry-pick/reset deliberately.
- Lost commits after bad rebase/reset/amend → they're in reflog for ~90 days. Deleted branch → `git reflog` or `git fsck --lost-found`.
- Uncommitted changes are the ONLY thing git can truly lose. Dropped stash → `git fsck --unreachable` and inspect the listed commit objects (works on any shell — no pipe needed). Never-added file → git can't help; check editor local history.

## Undo decision table

- Published/shared commit → **`git revert <sha>`** (new inverse commit; history intact; always safe).
- Local-only last commit, fix the message/content → `git commit --amend` (before pushing only).
- Local branch pointer wrong → `git reset --soft <sha>` (keeps changes staged) / `--mixed` (keeps in worktree) / `--hard` (DESTROYS worktree changes — user confirmation + stash first).
- One file back to a known state → `git restore --source=<sha> -- path` (surgical, nothing else moves).
- Never `reset --hard` or `push --force` as a reflex. Force-push only with explicit user OK and **`--force-with-lease`**, never bare `--force`.

## Bisect — find when it broke

```
git bisect start
git bisect bad                 # current is broken
git bisect good <known-good>   # last known good tag/sha
# git checks out midpoints; test each: git bisect good|bad
git bisect reset               # always, when done
```

- **Automate when a command can decide**: `git bisect run pnpm test -- --run path/to/repro.test.ts` — binary search does 1000 commits in ~10 steps while you get coffee.
- Prereq: a fast, deterministic repro. Flaky repro = lying bisect. Commits that don't build → `git bisect skip`.
- The guilty commit is the START of diagnosis, not the fix — read its diff, understand why, then fix forward.

## Conflicts & surgery tools

- Conflict resolution: understand BOTH sides before picking; `git log --merge -p -- <file>` shows the competing changes; after resolving, run the tests before continuing the rebase/merge. Repeated conflicts on long-lived branches → `git rerere` remembers resolutions.
- `git cherry-pick <sha>` — move a specific fix between branches (note: new SHA, plan for it at merge time).
- `git worktree add ../fix-branch <branch>` — second working copy for parallel work/bisect without stashing your state.
- `git stash -u` before risky operations; `git stash apply` (not `pop`) so the stash survives a botched apply.
- Whodunit: `git blame -w -C <file>` (ignores whitespace, follows moves), `git log -S "string"` (when did this string appear/vanish), `git log --follow -- path` (history across renames).

## Hard rules

Reflog checkpoint before surgery · revert over reset on anything pushed · `--force-with-lease` + explicit user OK, never bare force · `bisect reset` when done · destructive steps (reset --hard, clean, branch -D) get user confirmation every time — one approval ≠ standing approval.
