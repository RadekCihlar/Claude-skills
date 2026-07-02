---
name: repo-recon
description: Use when entering an unfamiliar codebase, module, or area of a repo BEFORE the first edit — maps entry points and conventions, and verifies that APIs/files/flags actually exist. Prevents hallucinated APIs, style mismatch, and reinvented utilities. Also use when about to call a library API you haven't verified in this project.
---

# Repo Recon

Goal: earn the right to edit. Time-boxed — recon is minutes, not a research project. Stop as soon as you can answer the four questions at the bottom.

## Sequence

1. **Manifest first.** `package.json` / `pyproject.toml` / `pom.xml` / `go.mod` / `Cargo.toml`: scripts (how to run/test/lint), dependencies (what's already installed — never re-add or reinvent), engines/toolchain pins.
2. **Orientation docs.** README, CLAUDE.md/AGENTS.md, CONTRIBUTING — skim for commands and gotchas only.
3. **Map the skeleton.** Glob top 2 levels. Identify: entry points, routing/wiring, domain logic, shared utils, tests location + naming pattern.
4. **Parallel search, not serial reads.** Fire Grep/Glob calls together: the symbol you'll touch, its callers, its tests, similar existing features. Read targeted ranges, not whole files.
5. **Neighbor rule.** Before writing anything, read 2-3 files that do a similar job. Capture: naming style, error handling, logging, import style, test structure. Your diff must be indistinguishable from theirs.
6. **Reuse sweep.** The util you're about to write almost certainly exists: grep for the concept (`formatDate`, `slugify`, `retry`, `clamp`) across `utils/ lib/ shared/ helpers/` before creating it.

## API truth verification (no hallucinated APIs)

Before calling any function/method you haven't verified IN THIS PROJECT:

- **Project code** → Grep the definition; read its actual signature.
- **Library** → check the installed version in the lockfile/manifest, then verify the API against `node_modules/<pkg>/` types (`.d.ts`), the package's README, or site docs for THAT major version. APIs move between majors — version first, docs second.
- **CLI flags / env vars** → `--help` output or source, never memory.
- Can't verify → say "unverified" and check, don't guess. A confident wrong call costs more than 30 seconds of Grep.

## Convention capture (fill before editing)

- Run/test/lint commands: `…`
- Naming: files `…`, symbols `…`
- Error handling style: `…`
- Test pattern + location: `…`
- Existing util that covers my need: `…` / none

## Done when you can answer

1. How do I run and test this? 2. Where does my change go and what wires it in? 3. What existing code/pattern do I imitate? 4. Which APIs am I calling and have I seen each one's real signature?

Can't answer one after honest searching → that's a question for the user, not a guess.
