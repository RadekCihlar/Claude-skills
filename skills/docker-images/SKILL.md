---
name: docker-images
description: Use when writing or editing a Dockerfile, docker-compose file, or building/optimizing container images — multi-stage builds, layer caching, non-root, signals, secrets, and tag discipline.
---

# Docker Images

An image is a deploy artifact: small, reproducible, non-root, and signal-correct. Most Dockerfile bugs ship silently and surface as prod incidents (PID 1 ignoring SIGTERM, secrets baked into layers, `:latest` drift).

## Build structure

- **Multi-stage always.** Build stage with the full toolchain → runtime stage with only the artifact + runtime deps. Compiled languages → distroless/slim runtime; JVM → JRE not JDK; Node → prune dev deps or bundle.
- **Layer order = cache order.** Least-volatile first: copy manifest + lockfile → install deps → THEN copy source. Copying source before install invalidates the dep cache on every code change.
- **Pin the base.** `node:22-slim`, `eclipse-temurin:21-jre` — a version, not `:latest`; digest-pin (`@sha256:…`) when reproducibility matters. Alpine = musl: verify native deps before choosing it.
- **`.dockerignore` is mandatory**: `.git`, `node_modules`/build dirs, `.env*`, local configs. Without it they leak into context and layers.
- Clean caches in the SAME `RUN` layer that created them (`apt-get install … && rm -rf /var/lib/apt/lists/*`) — a later `rm` doesn't shrink earlier layers.

## Runtime correctness

- **Non-root**: create a user, `USER app`. Required by hardened clusters, cheap everywhere else.
- **Signals / PID 1.** Exec-form `ENTRYPOINT ["app", "arg"]` — shell form wraps in `/bin/sh` which swallows SIGTERM → no graceful shutdown → k8s kills after grace period. App forks or won't reap zombies → `tini` as entrypoint.
- **One process per container.** Sidecars/orchestration belong to compose/k8s, not `&&` in CMD.
- `EXPOSE` documented port; healthcheck via orchestrator probes (k8s) or `HEALTHCHECK` (compose/swarm) — not both fighting.

## Secrets & config

- **Never** `COPY .env`, never secrets via `ARG`/`ENV` at build time — both persist in image history (`docker history` shows them). Build-time secret needed → BuildKit `--mount=type=secret`.
- Config at runtime via env vars / mounted files; the same image runs in every environment.

## Tags & shipping

- Immutable tags: git SHA or semver, promoted through envs. `:latest` in prod = "version: whatever". 
- Scan before ship when tooling exists (`trivy image`, `docker scout`); fix criticals in the base by bumping it, not by ignoring.
- Compose for local dev parity: same image, mounted source only for hot-reload dev targets.

## Checklist

Multi-stage, runtime stage minimal · deps cached before source copy · .dockerignore present · non-root USER · exec-form entrypoint, SIGTERM verified (`docker stop` exits fast, not after 10s timeout) · no secrets in any layer · pinned base · immutable tag.
