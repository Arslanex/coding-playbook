# 13 · Identity, cache, authz

WHEN: JWT, login/refresh/logout, password, Redis key for identity or cache, a permission check, or adding GraphQL/gRPC.
LOAD: this file **and** [15-security.md](15-security.md) — 15 owns the control list this file places on the tree.
RELATED: 10 (`http/deps` vs `modules/auth`) · 06 (models/repos) · 08 (cache client) · 05, 12 (401/403/404) — open only if the task is also that topic.
SCOPE: how identity and authorization sit on the backbone. Not tenant isolation (this playbook has no tenant).

Three different words that must not collapse into one folder:

- **DB session** — `AsyncSession` (06). One per request/job. Not a login.
- **Login session** — "this refresh token is still valid." Product: `modules/auth/`. Store: Postgres row and/or Redis denylist.
- **Cache** — reconstructible bytes in Redis (`infra/cache/`). Never the source of truth for an account.

---

## Where it goes

Stop at the first yes.

1. Sign/verify JWT, hash a password, "issue refresh", "revoke this login"?
   → `modules/auth/` (09). `http/deps` only **verifies** the access token (10).
2. Profile fields the user edits (`name`, avatar) as a product surface?
   → `modules/users/` when that folder would hurt if deleted. Until then: `AuthService` + `User` model.
3. "May this user cancel this order?"
   → owning **module service** (ownership) or `Depends(require_permission("…"))` when a permission table exists. MUST NOT [critical]: `if user.role` in the router.
4. GET/SET/TTL/lock against Redis, no product sentence if Redis disappeared?
   → `infra/cache/` (08). The **key and TTL** are chosen by the module (or by `http/` for rate-limit / idempotency / denylist).
5. GraphQL schema, gRPC proto, "flexible query for the frontend"?
   → **do not add** until a client that cannot use `/v1` exists. See Other transports.

MUST NOT: `shared/authz/`, `shared/jwt.py`, `infra/redis/`, `http/deps/auth.py` as the login product.

---

## JWT

Access token: short-lived (minutes, from `config/`). Claims: `sub` = `user_id`, `exp`, `iat`, `jti`. MUST NOT: permissions, email, roles that will go stale. MUST NOT: `tenant_id` unless the product has tenants.

Refresh token: longer-lived, rotated on use. Stored hashed (or as a row id + secret). MUST NOT [critical]: put the refresh token in logs or `details` (05).

`modules/auth` **signs**. `http/deps.get_current_user` **verifies** with the same secret/key from `config/` / secret store. External IdP (OIDC/SAML): [Extra 07](extra/07-sso.md) — still mint **our** JWT; MUST NOT: use the IdP access token as the API Bearer. MUST NOT: decode JWT in middleware "to rate-limit by user" if that forces a second decode — rate-limit by IP until identity has run, or reuse the already-verified `CurrentUser` in a later middleware. MUST NOT: a second JWT library.

Bearer in `Authorization`. Browser: access in memory; refresh in HttpOnly Secure cookie **or** rotated in the body — pick one and stay. MUST NOT [critical]: access token in `localStorage` as the playbook default (XSS reads it).

Public routes omit `get_current_user`. Login/register are public and **rate-limited** (10).

---

## Account (hesap)

`modules/auth/` day one: register, login, refresh, logout. Password hash **Argon2id** on `User.password_hash`. MUST NOT: plaintext, MUST NOT: reversible encryption.

Argon2id is CPU-bound by design — tuned correctly it costs tens of milliseconds **every** time. MUST: run hash and verify off the event loop (`asyncio.to_thread` or a thread executor). This is not conditional; a direct call blocks every other request in the process for the duration. Cost parameters from `config/` (03).

```
POST /v1/auth/register
POST /v1/auth/login      → access + refresh
POST /v1/auth/refresh
POST /v1/auth/logout     → revoke this login session
```

Issue tokens **after** `commit()` of the user/session row. Logout: revoke in Postgres (and denylist `jti` in Redis if access tokens must die before `exp`).

---

## Models (Postgres — source of truth)

Day one — only what login needs:

- `User` — `id`, `email` (unique), `password_hash`, timestamps (06 mixins). MUST NOT: `role` string "for later".
- `AuthSession` (or `RefreshToken`) — `id`, `user_id`, `token_hash`, `expires_at`, `revoked_at`, timestamps. One row per login device/session.

Add when the trigger is true (06: no unused columns):

- `email_verified_at` on `User` — when unverified users must not call protected routes.
- `PasswordReset` — `user_id`, `token_hash`, `expires_at` — when forgot-password exists. TTL short. Single use.
- `UserRole` + `Role` + `Permission` + `RolePermission` — when **two actor classes** need different verbs on the same resource (staff `orders.cancel` vs owner cancel). Until then: ownership in `OrderService` is enough.
- MUST NOT: `TenantMixin`. MUST NOT: GraphQL document tables. MUST NOT: store JWT access token as a row (use `jti` denylist in Redis, or keep access so short that logout only kills refresh).

Write owner: `AuthService` (and `UserService` if `modules/users/` exists). Other services **read** `UserRepository.get_by_id`; they MUST NOT update `password_hash`.

---

## Redis (`infra/cache/`)

Allowed (reconstructible):

- Rate-limit counters (10)
- Idempotency replay (10)
- Distributed lock (`lock:order:{id}` — **module** chooses the key)
- Access `jti` denylist until `exp`
- Hot read cache of a DTO the module already can reload from Postgres

MUST NOT: `User` as the only copy of the account. MUST NOT: permissions as the only copy. MUST NOT: job payload as the only copy of an order. If Redis is flushed, login still works from Postgres (refresh rows). Access tokens already issued stay valid until `exp` unless denylist existed and was lost — accept that or keep `exp` short.

Key: `{capability}:{purpose}:{id}` from the owning module, TTL explicit. MUST NOT: `infra/cache` invent `order:cancelled`.

`http/` may call `infra/` for rate-limit, idempotency, and identity verification (the `jti` denylist, and whether the account is still active). `infra/cache/` holds the reconstructible ones. A revocation list is not reconstructible — losing it un-revokes every token that was logged out — so it belongs beside the session rows in `infra/db/`, as *Done* below already says. Still MUST NOT: `infra/cache` import a module. MUST NOT: `http/` reach any of these through a module service — verification has no product sentence, so it has no business being a method on one (10).

---

## Authz

UI hiding a button is not authorization. Every write is checked on the server.

Day one (no permission tables):

- Protected route: `Depends(get_current_user)`.
- Resource: `OrderService.get/cancel(order_id, user_id)` — missing and not-owned → same 404 (05, 12).
- MUST NOT: 403 on "not your order". MUST NOT: `if current_user.id == …` in the router.

When permission tables exist:

- `Depends(require_permission("orders.cancel"))` on staff routes. Implementation: load grants for `user_id` (Postgres; optional cache). MUST NOT: trust a `perm` list in the JWT as the only check.
- Ownership routes still use the service, not `require_permission`.
- `require_permission` lives next to `http/deps` if it only reads grants + raises `AuthorizationError`. The **catalog** of permission strings and who assigns them is `modules/auth/` (or `modules/rbac/` if that is its own ability). MUST NOT: Casbin/OPA/graph library until a second engine is real.

Workers: payload has `user_id` when the actor matters; the **service** still enforces. MUST NOT: skip ownership because "it came from the queue".

---

## Other transports (GraphQL / gRPC)

MUST NOT: add GraphQL or gRPC because the frontend wants fewer round-trips. `/v1` + module routers is the public API (12).

If a **new client class** cannot speak REST (native mobile batch, mesh service):

- GraphQL — another transport shell, not a second product tree. Schema maps to the same module **services**. MUST NOT: resolvers → repositories. MUST NOT: a parallel `modules/` for GraphQL. Mutations still raise the same errors; map them like `http/errors/` (one JSON/status story, or GraphQL errors that still carry `error_code`).
- gRPC — `rpc/` next to `http/` if it is a second process protocol. Same services. MUST NOT: put proto stubs in `infra/`.

Until that client exists: zero files. A `graphql/` folder "for later" is noise.

---

## Backend security (this slice)

The control list is [15-security.md](15-security.md) — it is not restated here, so there is one place to change it. This section only names what is specific to **identity**:

- Login failure message is generic. MUST NOT: reveal which of email/password was wrong.
- Minimum password length enforced on the server, never only in the form.
- Hash and verify run off the event loop (Account, above).
- `audit` on login, logout, password change, permission grant (04).
- The password never reaches a log, an exception `message`, or `details` (05).
- A revoked refresh row is checked on every refresh. An access token stays valid until `exp` unless the `jti` denylist exists.

Everything else on a protected route — rate limit, CORS, parameterised SQL, request schema, error body, secret handling — is 15's numbered layer list. Skipping one is a defect there.

---

## Done

- [ ] Login/issue/revoke in `modules/auth/`; `http/deps` only verifies
- [ ] The verify path reads `infra/`, not a module service — signing is a product act, verifying is a lookup
- [ ] `User` + session/refresh row in Postgres; Redis holds only reconstructible bytes
- [ ] Access JWT is short, claims are `sub`/`exp`/`jti` — not a permission dump
- [ ] Resource authz is the owning service (404); staff verbs need tables before `require_permission`
- [ ] No GraphQL/gRPC folder without a client that cannot use `/v1`
- [ ] Password Argon2id, hashed off the event loop; no token in logs; rate-limit on auth routes
- [ ] 15's layer list was run for this change
