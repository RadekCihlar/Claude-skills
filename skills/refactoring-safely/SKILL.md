---
name: refactoring-safely
description: Use before restructuring working code — renames, extractions, moving modules, replacing implementations, or any "rewrite this properly". Behavior-preserving discipline that keeps the diff reviewable and the system green throughout.
---

# Refactoring Safely

Refactoring changes structure, never behavior. The moment you "improve" behavior mid-refactor, you have two risky changes wearing one commit.

## Before touching anything

1. **Green baseline.** Run the relevant tests; record the pass. Refactoring on red = flying blind.
2. **Coverage check on the target.** Behavior about to be restructured but untested → write **characterization tests first**: pin down what the code ACTUALLY does now (including its weird cases), not what it should do. They are your safety net.
3. **Scope sentence.** "Extract X so Y" — write it, stay inside it. Adjacent mess gets a `⚠️ Tech debt:` note, not a drive-by.

## During

- **Small reversible steps, green between each.** Rename → run tests → extract → run tests. Never a 40-file big-bang where step 3's breakage hides inside step 9's diff.
- **Mechanical transforms via tooling** (IDE/LSP rename, codemod) over hand-editing every call site — tools don't get bored on file 27.
- **Separate commits: refactor vs behavior.** If a behavior change is needed, finish the refactor, commit, then change behavior in its own commit. A reviewer must be able to skim the refactor commit trusting "no behavior change".
- **Keep the API stable from the outside** unless changing it IS the task; then update every caller in the same change — no half-migrated states left behind.
- Don't refactor and reformat together; formatting noise buries the real diff.

## Replacing something big (rewrite territory)

- **Strangler pattern over big-bang**: new implementation grows beside the old behind the same interface; callers migrate incrementally (feature flag / percentage / per-route); old one dies only when nothing calls it.
- Old and new coexisting must both pass the same test suite — run the suite against both if the seam allows.
- A rewrite with no incremental path needs explicit user sign-off; it's the highest-risk move in software.

## Stop conditions

- Tests red for more than one small step → revert to last green, take a smaller step.
- Scope creeping ("while I'm here…") → stop, note it, finish the stated scope.
- "Improvement" makes the code longer AND harder to explain → the old code was fine; revert.

## Done means

Same observable behavior (tests prove it) · every step was green · refactor isolated from behavior changes in history · no orphaned/half-migrated callers · scope sentence satisfied, nothing more.
