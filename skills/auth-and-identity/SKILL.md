---
name: auth-and-identity
description: Use when building or reviewing login, sessions, JWTs, OAuth2/OIDC flows, password handling, RBAC/permissions, CSRF protection, or any "who is this and what may they do" logic.
---

# Auth & Identity

Rule zero: **never roll your own auth protocol or crypto.** Use the platform's mature library (Spring Security, NextAuth/Auth.js, Keycloak, the framework's session layer). Your job is wiring it correctly, and that's hard enough.

## Authn vs authz — keep them distinct

Authentication = who you are. Authorization = what you may do. Every endpoint needs both answered, and authz is checked **per resource**: "logged in" + "owns order 123" — the missing second check is IDOR, the most common real-world vuln. Deny by default; server-side always (client checks are UX, not security).

## Sessions vs JWT — pick for revocation, not fashion

- **Default for web apps: server-side sessions** in an httpOnly cookie. Instantly revocable, small, boring, battle-tested.
- **JWTs earn their place** for service-to-service and stateless APIs at scale. Cost: revocation is hard — so access tokens SHORT-lived (minutes), paired with rotating refresh tokens, and a "kill this user's access now" story decided BEFORE shipping (denylist, token version claim, short expiry). No revocation story = no JWTs.
- Validate everything on every request: signature, `exp`, `iss`, `aud`, algorithm pinned (reject `none`/alg-swap). Access token ≠ identity — identity comes from the ID token / userinfo (OIDC).
- Never put secrets or sensitive PII in a JWT — it's readable by anyone who holds it.

## Cookies & CSRF

- Auth cookies: `HttpOnly` (no JS access), `Secure`, `SameSite=Lax` minimum.
- Cookie-authenticated state-changing requests need CSRF protection: SameSite helps but framework CSRF tokens remain the standard for POST/PUT/DELETE from browsers. Token-in-header APIs (no cookie auth) are CSRF-immune by construction.

## Passwords

argon2id or bcrypt via the library — never fast hashes (SHA-*, MD5) even salted. Allow long passwords (64+ chars) and paste; no composition-rule theater. Rate-limit login per account AND per IP; be careful lockouts don't become a DoS lever. Password reset = the second front door: single-use expiring tokens, no user enumeration in responses ("if the account exists, we sent a mail").

## OAuth2 / OIDC

User-facing flows: **authorization code + PKCE**, always. Implicit flow is dead; resource-owner-password is dead. Validate `state` (CSRF) and `nonce` (replay). Request minimal scopes. Third-party IdP ("login with X") → the `sub` claim is the stable identity key, never the email (emails change and get recycled).

## Authorization model

Coarse roles + fine permissions checked at the resource ("can edit THIS doc"). Centralize the check (middleware/policy layer) — scattered inline checks drift. Admin/impersonation paths get extra logging. Audit log auth events: login success/fail, password/permission changes, token issuance — that's incident forensics.

## Checklist

Library not hand-rolled · authz per resource, deny-by-default · revocation story exists · cookies HttpOnly+Secure+SameSite, CSRF covered · argon2id/bcrypt + rate limits · PKCE + state + nonce · sub not email as identity key · auth events audited.
