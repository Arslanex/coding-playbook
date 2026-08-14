# 12 · Auth (cookies, not JWT-as-UI)

WHEN: sign-in/out, a protected route, session cookie, or "the client must know who is logged in."
LOAD: this file. Backend identity: [python-fastapi-backend/13-identity-security.md](../python-fastapi-backend/13-identity-security.md) when the cookie/JWT split is in doubt. Security: 15.
RELATED: 06 (redirect) · 09 (401) — open only if the task is also that topic.
SCOPE: how this app **carries** the session. It does not issue tokens and does not enforce "may cancel."

FastAPI signs. This app sends the cookie and renders. Authz is still the API (404 for not-owned).

---

## Cookie

MUST: session/refresh in HttpOnly + Secure + SameSite cookie the API Set-Cookie'd (or the BFF copied). The browser does not read the token for logic.

MUST NOT: access JWT in `localStorage` or `sessionStorage`.
MUST NOT: `jwt-decode` to show permissions or skip a route. Claims go stale; the API is the gate (backend 13).
MUST NOT: `Authorization: Bearer` from a token the JS bundle holds, unless Extra says the product has no cookie (then memory only, never localStorage — backend 13).

Middleware (`proxy.ts` / `middleware.ts`): may redirect to login when the **cookie is missing**. MUST NOT: treat a present cookie as "this user may open /admin" — still fetch or trust a server layout that asked the API.

---

## `features/auth`

Login/register/logout pages. Forms POST to FastAPI (08). After login: redirect; server reads follow with the cookie (07).

A small `CurrentUser` on the server (from API `/v1/auth/me` or equivalent) for display names. MUST NOT: a global Zustand user as the source of truth (13).

---

## Done

- [ ] HttpOnly cookie; no token in localStorage
- [ ] No client JWT permission checks
- [ ] Middleware only catches "no cookie"; API still 401/404
