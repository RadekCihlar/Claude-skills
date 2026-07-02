---
name: testing-strategy
description: Use when deciding WHAT to test and at which level (unit/integration/e2e), structuring test files, fixing flaky tests, or judging coverage — the strategy layer that complements the TDD red-green-refactor loop.
---

# Testing Strategy

Tests exist to let you change code without fear. Optimize for failure clarity: when this test goes red, does the reader know instantly what broke?

## What to test at which level

- **Unit** — pure logic, branching, edge cases, error paths. Fast, no I/O. The bulk of tests live here because this is where the branches live.
- **Integration** — the seams: DB queries against a real (containerized/local) DB, HTTP handlers through the real router + middleware, serialization round-trips. Fewer, slower, catch what mocks hide.
- **E2E** — a handful of golden paths through the real stack. Expensive and flaky-prone; every one must earn its place.
- Push each behavior to the LOWEST level that can catch its regression.

## What NOT to test

- Framework behavior, language features, third-party internals.
- Private functions directly — test through the public surface; needing to test a private is a design smell.
- Exact copy/markup snapshots that fail on every harmless change — assert on behavior and semantics (roles, labels, outputs), not incidental structure.
- Mock-echo tests that only verify you called the mock the way you mocked it — they test the test.

## Test construction

- **Arrange-act-assert, one behavior per test.** Multiple asserts fine when they describe one outcome.
- **Name = specification**: `rejects expired token`, `retries 3 times then surfaces error` — readable as a spec sheet, no `test1`/`works`.
- **Minimal fixtures.** Build the smallest input that exercises the branch; builder/factory helpers over 50-line shared fixtures every test depends on. Shared mutable fixtures = coupled tests.
- **Determinism.** No real clock (`Date.now` → fake/injected), no real randomness (seed it), no real network (fake at the boundary), no sleeps — await the actual condition or advance fake timers.
- **Boundaries + error paths beat more happy paths**: empty, one, many, max, malformed, unauthorized, upstream-down.
- Mock at architectural boundaries you own (the HTTP client wrapper, the repo interface), not deep internals — deep mocking welds tests to implementation.

## Flaky tests

A flaky test is a bug, not weather. Quarantine fast, then root-cause: shared state between tests, order dependence, real time/network, unawaited async, port collisions. Fix or delete — a test nobody trusts is worse than none. Never "fix" by adding retries or sleeps.

## Coverage

Coverage is a flashlight, not a target. Use it to find untested BRANCHES in code you touched; never write assertion-free tests to move a number. New logic → its branches covered; that's the bar.

## Regression rule

Every bug fix ships with the test that would have caught it, placed at the lowest level that reproduces it — that test is the fix's proof and its tombstone.
