# Extra 07 · SSO (OIDC / SAML)

WHEN: users sign in through an **external IdP** (Google, Okta, Entra, a customer SAML) instead of (or in addition to) email+password.
LOAD: this file **and** [10](../10-http.md), [13](../13-identity-security.md), [15](../15-security.md). Not instead of them.
SCOPE: how SSO attaches to `modules/auth/` + `infra/`. MUST NOT: a new identity backbone. MUST NOT: `src/sso/` / `http/oidc.py` as the login product.

Default remains 13: `AuthService` issues **this** API's JWT. SSO proves who the human is. It does not replace `get_current_user`.

---

## Decide protocol

Stop at the first yes.

1. IdP speaks OIDC / OAuth 2 authorization code?
   → OIDC. `infra/oidc/` (or `infra/idp/` while there is one vendor file).
2. IdP only speaks SAML 2?
   → SAML. Same folder, second vendor file when both exist (08). MUST NOT: SAML "because enterprise" if OIDC is offered.
3. Password-only, no IdP?
   → **not this Extra**. Stay on 13.

MUST NOT: implicit flow. MUST NOT: IdP access token as the API Bearer (stale claims, wrong audience). After the callback, `AuthService` mints **our** access + refresh (13).

---

## Where it sits

```
modules/auth/              # start SSO, callback, link account, issue our JWT
infra/oidc/ (or idp/)      # redirect URL, code exchange, fetch claims. No "user may login".
http/deps                  # still only verifies our JWT (10)
```

Routes (public, **rate-limited** — 10, 15):

```
GET  /v1/auth/sso/{provider}/start      → 302 to IdP
GET  /v1/auth/sso/{provider}/callback   → our tokens (or 302 to the app with a one-time code)
```

MUST NOT: put the IdP SDK in `http/` or in `core/`. MUST NOT: `Authlib` as a folder name.

`config/`: client id, redirect URI, issuer URL. Secrets in the secret store. MUST NOT: client secret in the frontend.

---

## Account link

Postgres stays the source of truth (13).

- `User` — still `id`, email, timestamps. `password_hash` **nullable** when this user has no password.
- `UserIdentity` — `user_id`, `provider`, `subject` (IdP `sub`), unique (`provider`, `subject`). Write owner: `AuthService`.

First login: create `User` + `UserIdentity` in one commit, then issue tokens. Later login: lookup identity, rotate refresh.

Email from the IdP is a hint. MUST: verify the IdP's ID token signature and `aud` / `iss` / `nonce` (or SAML signature + audience) in `infra/` before `AuthService` trusts `sub`. MUST NOT: trust email query params.

If email matches an existing password user: product policy on the **service** (auto-link vs "set password / confirm"). MUST NOT: silent link without a rule. [Extra 01](01-multi-tenant.md): membership is still `TenantMembership`; IdP `tid` is not `get_tenant` (tenant from URL).

Logout: revoke **our** `AuthSession` (13). MUST NOT: skip because "IdP session remains" unless the product also needs RP-initiated logout — then `AuthService` calls `infra/` after revoke.

---

## What not to put in the JWT

Our access token claims stay `sub` / `exp` / `iat` / `jti` (13). MUST NOT: copy IdP groups into the access JWT as the only authz. Staff verbs: permission tables when they exist. Resource authz: owning service.

---

## Done

- [ ] Callback in `modules/auth/`; IdP HTTP in `infra/`; `http/deps` still verifies our JWT
- [ ] `UserIdentity` links `provider`+`sub`; password may be null
- [ ] Code flow + signature checks; no IdP token as API Bearer
- [ ] Start/callback rate-limited; secrets not in the frontend
