---
name: api-design
description: Use when designing or extending any programmatic interface — HTTP/REST endpoints, RPC, webhooks, or a library's public API — or writing an OpenAPI/Swagger spec or API documentation. Covers naming, error shape, pagination, idempotency, versioning, and compatibility rules.
---

# API Design

An API is a promise. Design for the caller you can't see, and assume every published behavior will be depended on (Hyrum's law).

## HTTP endpoints

- **Resources, not verbs.** `GET /orders/{id}`, `POST /orders`, not `/getOrder`. Actions that don't map → sub-resource verb (`POST /orders/{id}/cancel`), used sparingly.
- **Status codes carry meaning.** 200/201/204 success; 400 malformed; 401 unauthenticated; 403 unauthorized; 404 missing (also for "exists but you may not know"); 409 conflict; 422 semantic validation; 429 rate limited (+ `Retry-After`); 5xx = our fault, never for caller errors.
- **One error shape everywhere.** RFC 9457 problem+json or the project's existing shape: machine-readable `code`, human `message`, optional `field` errors. Never leak stack traces or internals outward.
- **Validate at the edge with a schema** (Zod/etc.); reject unknown fields or ignore them — pick one policy and keep it.
- **Pagination: cursor over offset** for anything that grows (offset skews under writes and gets slow deep in). Response carries `next_cursor`; page size capped server-side. Always paginate collections — "return all" is a future incident.
- **Idempotency.** GET/PUT/DELETE idempotent by design. POST that creates → accept `Idempotency-Key` when retries are possible (payments, jobs). Retried request returns the original result, not a duplicate.
- **Timeouts + limits are part of the contract.** Request body size cap, rate limits documented, slow endpoints get async job + status polling instead of 5-minute requests.

## Webhooks

Sign payloads (HMAC + timestamp, reject stale), deliver at-least-once → receivers must be idempotent, retry with backoff, include event `id` + `type` + versioned payload.

## Compatibility & versioning

- **Additive changes are free** (new optional field, new endpoint). Breaking = removing/renaming fields, changing types/semantics, tightening validation on existing input.
- Breaking change → new version (path `/v2` or header) or new field alongside old + deprecation window. Never silently change behavior of an existing endpoint.
- Clients must tolerate unknown fields in responses — state it in the contract.

## Library/public APIs

- Smallest surface that serves the use case; everything exported is forever.
- Accept the general type, return the specific one. Options object over 4 positional args.
- Errors: typed/documented, thrown for exceptional, encoded in return for expected (match language idiom).

## Checklist before shipping

Naming consistent with existing endpoints · error shape matches project standard · collections paginated · auth + authz per resource · input schema at boundary · idempotency story for retries · breaking-change check against current callers.
