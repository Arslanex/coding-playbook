# 15 · Security (never skip)

WHEN: any PR, new route, auth change, upload, webhook, or config/secret. Load **before** merge, not after a finding.
LOAD: this file only. It is the canonical control list — 13 places identity on the tree, it does not restate these.
RELATED: 13 (identity placement) · 05, 12 (error JSON) · 10 (HTTP shell) · 06 (SQL) · 04 (logs) — open only if the task is also that topic.
SCOPE: controls that must exist on a public backend. Skipping a layer because "the frontend checks it" or "we'll add it later" is a defect.

A hidden button is not a control. A comment is not a control. A layer is code that still runs if the client is curl.

---

## Layers on a request (do not remove one)

Stop and restore if any layer is missing on a **protected** route.

1. **Config / secrets** — signing keys, DSN, SMTP from `config/` + secret store. `.env` not in git. MUST NOT: secret in frontend bundle, test fixture copied from prod, or CI log.
2. **Ingress** — CORS explicit origins when browsers exist (MUST NOT: `*`, especially with credentials). TLS at the platform. `/docs` off in production (`config/`).
3. **Rate limit** — `http/middleware` + `infra/cache` (10). MUST: login, register, refresh, password-reset. MUST: any new brute-force surface (OTP, invite redeem). Anon: IP. After identity: also `user_id`.
4. **Identity** — `Depends(get_current_user)` on protected routes (10, 13). No/invalid Bearer → 401 `UNAUTHENTICATED`. Public list is explicit, not the default.
5. **Authz** — owning **service** checks ownership (13). Missing and not-owned → **same 404** (05, 12). Staff verb → `require_permission` only after permission tables exist. MUST NOT: `if user.role` in the router. MUST NOT: 403 that confirms the id. Workers use the same service — MUST NOT: skip because "it came from the queue" (11).
6. **Validation** — Pydantic request schema (12). `extra` forbidden or PATCH `exclude_unset`. MUST NOT: `**body` onto the ORM. `id`, `password_hash`, `created_at` are not writable from the client.
7. **Persistence** — UUID PK (06). Parameterised SQL only. MUST NOT: f-string SQL. Cache is reconstructible (13); account truth is Postgres.
8. **Errors** — only `{error_code, message, details}` (12). MUST NOT: stack, SQL, DSN, token in `details` or `message`. FastAPI `debug` off in production.
9. **Audit** — login, logout, password change, permission grant, money (`LoggerName.AUDIT`, 04). Passwords never in `extra=`.

---

## Always on (day one of a real API)

MUST:

- Argon2id on `password_hash`. Never in `UserResponse`. Login failure message generic (do not say which of email/password failed). Min length enforced **on the server**.
- Access JWT short; claims `sub` / `exp` / `jti` only (13). Refresh rotated and revoked on logout. MUST NOT: `?token=` in a URL. MUST NOT: access token in `localStorage` as the default (13).
- `X-Request-ID` echoed (12). 5xx stack in logs only (`LoggerName.ERROR`).
- Production `LOG_LEVEL` is not DEBUG.

MUST NOT:

- Admin / staff routes without identity + authz.
- Sequential integer public ids.
- Process-local "logged-in users" map (02).
- Trusting the client for `user_id` in the body when the actor is the Bearer subject.

---

## Add when that feature exists (do not ship the feature without the control)

Upload (`infra/storage/`, 08): auth + ownership; max size from `config/`; MIME from magic bytes (not only extension); sanitised name; object storage not Postgres BYTEA; short signed URL; no token in the upload query string.

Inbound webhook: HMAC verify (`compare_digest`) + timestamp window **before** the service runs. Signing secret in the secret store. Outbound: sign the body; MUST NOT: put secrets in the payload.

Email verification: column + one-time token when unverified users must not perform the sensitive action. MUST NOT: silently skip.

Password reset: short-lived hashed token, single use, rate-limited.

---

## Agent pass (PR)

Every item true or the change does not merge:

- [ ] No `.env` / prod secret in the diff
- [ ] No secret in frontend
- [ ] Protected route has `get_current_user` (or documented public)
- [ ] Write path: service ownership or `require_permission` — not UI-only
- [ ] Auth-adjacent route is rate-limited
- [ ] SQL parameterised; no string-built query
- [ ] Request schema; no `**body` onto the model; no `password_hash` on the response
- [ ] 404 for missing and not-owned; error body has no stack/DSN/token
- [ ] CORS not `*` (if this change touches origins)
- [ ] Upload/webhook/admin: matching control in "Add when" above is present
- [ ] Test exists for the 401 / 404-not-owned / validation case that this route introduced (14)

---

## Done

- [ ] Did not skip a numbered layer on a protected route
- [ ] Did not add a feature from "Add when" without its control
- [ ] Agent pass is all yes
