---
name: http-networking
description: Use when dealing with CORS errors, cookies across origins, TLS/certificates, reverse proxies and X-Forwarded-For, load balancer timeouts, DNS, connection pools, or mysterious 502/504s.
---

# HTTP & Networking

The layer everyone assumes "just works" until a proxy header, a timeout mismatch, or an expired cert takes prod down.

## CORS — browser-only, misunderstood daily

- CORS protects USERS from malicious sites, not your API from clients — it is not authorization. curl ignores it entirely.
- Fix it server-side: explicit origin allowlist. **Never `Access-Control-Allow-Origin: *` together with credentials** (spec forbids it; wildcards + cookies don't mix). Echoing the request Origin unchecked = same as `*`.
- Preflight: non-simple requests trigger `OPTIONS` first — your routes/middleware must answer it (headers + methods allowed) or every request "fails CORS" while the server never saw the real call. Cache it: `Access-Control-Max-Age`.
- The error in the browser console names the missing piece — read which header it wants; don't shotgun headers.

## Proxies & X-Forwarded-For

- Behind any proxy/LB, `remoteAddr` is the proxy, not the client. Client IP = the **rightmost XFF entry added by a proxy you trust** — the header is client-spoofable, so trust only your known hop count/hosts. Rate limiting, audit logs, and geo checks are wrong (or bypassable) if this is wrong.
- Forwarded proto/host (`X-Forwarded-Proto/Host`) drive redirect and URL generation behind TLS-terminating proxies — misconfigured = redirect loops or http URLs in a https app.

## Timeouts — align the chain or eat 502/504s

Every hop has a timeout: client → CDN → LB → app server → upstream. **Each layer's timeout must be shorter than the layer in front of it**, or the outer layer gives up first and returns 502/504 while your app happily finishes work nobody receives. Idle/keep-alive mismatches cause intermittent connection-reset errors: app's keep-alive timeout should EXCEED the LB's idle timeout (or the LB reuses a connection the app just closed). Long work (> LB timeout) → async job + polling, not a longer timeout arms race.

## TLS & DNS

- Terminate TLS at the edge, redirect http→https (301), HSTS once confident (start small max-age).
- **Cert expiry is a monitored metric with alerting** — it is the most preventable outage in the industry. Automate renewal (ACME) and still monitor.
- Internal traffic isn't automatically trustworthy — mTLS or network policy where the threat model warrants.
- DNS TTLs bound your failover speed — 24h TTL means a day of clients hitting the dead IP. Don't hardcode IPs; don't cache DNS forever in long-lived processes (JVM historically caches successful lookups indefinitely under a security manager — check `networkaddress.cache.ttl`).

## Connections & payloads

- Reuse connections: keep-alive/HTTP2, pooled clients created ONCE (per-request client creation = handshake tax + port exhaustion). Pool sized to downstream capacity, not "big".
- Compress text responses (gzip/brotli); never double-compress images/video. Cap request body sizes at the edge.
- Retries exist at multiple layers (client lib, service mesh, LB, your code) — they multiply. Budget retries in ONE place; see error-resilience.
- WebSockets/SSE through proxies need explicit idle-timeout and upgrade support — the default LB config kills them silently at 60s.

## Checklist

CORS allowlist server-side, preflight handled · XFF trusted-hop parsing for anything security-relevant · timeout chain aligned outer>inner, keep-alive app>LB · cert renewal automated AND alerted · pooled clients, bounded bodies · retries budgeted once.
