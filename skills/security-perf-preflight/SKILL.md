---
name: security-perf-preflight
description: Use BEFORE writing or editing code that touches a trust boundary (user input, HTTP handlers, DB queries, file paths, shell commands, auth, uploads, redirects) or a hot path (loops over requests/frames, timers, caches, large collections). Applies the security + performance stance up front instead of bolting it on after.
---

# Security & Performance Preflight

Run this mentally before the first line, not after review finds it. One pass, ~30 seconds. Anything that fails a check either gets designed around now or flagged as `⚠️ Tech debt:` one-liner.

## Security — by trust boundary

Identify every place data crosses from untrusted to trusted. For each:

- **Injection.** SQL → parameterized queries only, never string-built. Shell → avoid spawning with user input; if unavoidable, pass args as array, never interpolate into a command string. HTML → escape by default; framework escaping stays on; `dangerouslySetInnerHTML`/`innerHTML` needs written justification + sanitizer.
- **Path traversal.** User-supplied names never concatenated into paths. Resolve then verify the result is inside the allowed root (`path.resolve` + prefix check). Reject `..`, absolute paths, null bytes.
- **SSRF.** User-supplied URLs fetched server-side → allowlist hosts/schemes, block private IP ranges (127.0.0.0/8, 10/8, 172.16/12, 192.168/16, 169.254/16, ::1), re-check after redirects.
- **Authz ≠ authn.** Every handler that reads/writes a resource checks the caller owns/may access THAT resource (IDOR). Logged-in is not authorized.
- **Validation at the boundary, once.** Schema (Zod/etc.) at the edge → trusted types inside. Don't re-validate everywhere; don't trust client-side validation at all.
- **Secrets.** Never in logs, error messages, URLs, client bundles, or committed files. Env vars / secret manager only. Error responses: generic outward, detailed in server logs.
- **Crypto.** Never hand-rolled. Passwords → argon2/bcrypt. Tokens → crypto-random, not Math.random. Compare secrets with constant-time compare.
- **Deserialization / prototype pollution.** No `eval`, no deserializing untrusted blobs into rich objects; JSON.parse then validate shape. Watch `__proto__`/`constructor` keys in merges.
- **Open redirect.** Redirect targets from params → allowlist relative paths or exact hosts.
- **Uploads.** Validate type by content not extension, cap size, store outside webroot, never execute.

## Performance — by budget

Ask: how often does this run × how much does it cost? Then:

- **Hot path = O(n) discipline.** No O(n²) scans, no sync I/O, no JSON.parse/stringify of large blobs per request/frame. Move invariants out of loops.
- **N+1.** Query in a loop = redesign now (batch, join, dataloader). It never gets fixed later.
- **Bound everything.** Caches get max size + eviction. Collections that only grow are leaks. Timers/listeners are cleared on unmount/shutdown.
- **Animation/polling.** Cap fps, pause on `document.hidden` and offscreen. Poll intervals justified; prefer push/stream when available.
- **Payloads.** Paginate lists; never "fetch all then filter client-side" for unbounded data. Compress/cache static responses.
- **Concurrency.** Independent awaits → `Promise.all`. But bound parallelism against upstreams with rate limits.
- **Measure before optimizing anything cold.** Preflight covers structural waste; micro-optimization waits for a profiler.

## Output contract

- Designed-in mitigations: silent — that's just the code.
- Risks found in EXISTING nearby code: `⚠️ Tech debt:` one line each, don't fix unasked.
- A check you deliberately skipped (impossible input, internal-only surface): state the assumption in one line.
