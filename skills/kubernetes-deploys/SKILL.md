---
name: kubernetes-deploys
description: Use when writing or editing Kubernetes manifests/Helm charts, configuring probes/resources/rollouts, or debugging pods (CrashLoopBackOff, OOMKilled, Pending, ImagePullBackOff, unready endpoints).
---

# Kubernetes Deploys

k8s does exactly what the manifest says, including the wrong thing, at scale. Probes and resources are not boilerplate — they ARE the availability story.

## Probes — the #1 source of self-inflicted outages

- **Readiness** = "send me traffic": checks the app can serve (dependencies optionally included). Fails → pod removed from Service endpoints, NOT restarted.
- **Liveness** = "restart me": checks the PROCESS is alive (deadlock detection), must NOT check downstream dependencies — DB blip + liveness-checks-DB = restart storm across the fleet amplifying the outage.
- **Startup** for slow boots (JVM): buys time before liveness kicks in; better than a 5-minute `initialDelaySeconds`.
- Probe endpoints are cheap, unauthenticated, no side effects.

## Resources

- **Requests always set** — they drive scheduling and define QoS. No requests = BestEffort = first evicted.
- **Memory: request = limit** (or close) — memory isn't compressible; overcommit = random OOMKills. JVM/Node: heap flag coherent with the limit (`-XX:MaxRAMPercentage`, `--max-old-space-size`), or the runtime OOMs the container from inside.
- **CPU: set request, consider skipping the limit** — CPU throttling on limit hits latency hard; measure before capping.
- HPA needs requests to compute utilization; don't autoscale JVMs on memory (heap never shrinks).

## Rollouts & availability

- RollingUpdate with `maxUnavailable: 0, maxSurge: 1` for small deployments; `progressDeadlineSeconds` so a broken rollout FAILS visibly instead of hanging.
- **Graceful shutdown chain**: SIGTERM → app stops intake, drains in-flight → exits < `terminationGracePeriodSeconds`. Add `preStop: sleep 5` so the endpoint removal propagates to kube-proxy/LBs before the process stops — otherwise brief 502s on every deploy.
- **PodDisruptionBudget** for anything with >1 replica — node drains respect it; without it maintenance can take all replicas at once.
- Replicas ≥2 + `topologySpreadConstraints`/anti-affinity across nodes/zones, or one node failure = outage.

## Config & security

- ConfigMap/Secret via env or mount; Secret ≠ encrypted (base64) — RBAC-restrict it, consider external-secrets/SOPS for the source of truth. Never secrets in image or manifest repo.
- `securityContext`: `runAsNonRoot`, `readOnlyRootFilesystem`, `allowPrivilegeEscalation: false`, drop ALL capabilities, add back only needed.
- Image by immutable tag; `imagePullPolicy: IfNotPresent` with immutable tags.

## Pod triage decision tree

- **Pending** → `kubectl describe pod`: insufficient resources (requests too big / cluster full), unschedulable (taints, affinity), or PVC unbound.
- **ImagePullBackOff** → image name/tag typo, registry auth (`imagePullSecrets`), or rate limit.
- **CrashLoopBackOff** → `kubectl logs --previous` (the crash is in the PREVIOUS container); config/env missing, migration failed, port bound.
- **OOMKilled** (`describe` → Last State) → raise limit only after checking for a leak/heap-flag mismatch.
- **Running but 0/1 Ready** → readiness probe failing: `describe` shows probe error; check the endpoint manually via `kubectl exec`/`port-forward`.
- **Service unreachable** → `kubectl get endpoints`: empty = selector/label mismatch or nothing ready; check targetPort matches containerPort.
- Live inspection: `kubectl exec -it`, ephemeral `kubectl debug` for distroless, `kubectl get events --sort-by=.lastTimestamp`.

## Checklist

Readiness ≠ liveness, liveness dependency-free · requests set, memory req=limit, heap flags coherent · graceful SIGTERM + preStop drain · PDB + spread for HA · progressDeadline set · secrets RBAC'd, never in repo · securityContext hardened.
