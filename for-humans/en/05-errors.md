# 50 common vibe-coding errors

> **For humans** — a quick-reference checklist for when an AI-built app breaks.

Playbook stacks: **Next.js UI** + **FastAPI API**. Fixes below assume that split unless noted.

**Root causes are predictable, not random:** context lost between files · deprecated APIs in training data · happy-path-only generation · no system-wide view. A large share of AI code gets reverted within weeks — usually because it never matched the **system**, not because every line was wrong.

| Category | Count | Share |
|----------|------:|------:|
| Build & compilation | 10 | 20% |
| Runtime & logic | 15 | 30% |
| API & data | 10 | 20% |
| Auth & authorization | 8 | 16% |
| Deployment | 7 | 14% |

Runtime dominates because agents skip edge cases ([04-pitfalls §9](04-pitfalls.md)).

**Türkçe:** [05-errors.md](../tr/05-errors.md) · **Index:** [for-humans/](../README.md) · **Agents read:** [agents/README.md](../../agents/README.md)

---

## How to use this page

1. Copy the **exact** error text from terminal, browser, or CI.
2. Find the number below (or search this file).
3. Apply the fix; `@` the playbook file in your next agent prompt.
4. **Do not** prompt “fix my app” without the error message ([02 How to prompt](02-how-to-prompt.md)).

**Agent prompt for any error:**

```text
Stack: [nextjs-frontend | python-fastapi-backend]
Error (verbatim): [paste]
File / line: [path:line]
Task: Fix only this error; minimal diff.
Playbook: [relevant numbered file from fix column]
Done when: error gone; tests pass; no unrelated refactor
```

---

## A · Build & compilation (1–10)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 1 | **Module not found** (npm/pip) | Install the package; lock the version in `package.json` / `pyproject.toml`. | — |
| 2 | **Cannot find module `./X`** | Verify path and **case** (Linux CI is case-sensitive). | [nextjs 03](../../nextjs-frontend/03-file-structure.md) |
| 3 | **Property does not exist** (TS) | Match the type/API; do not `as any` to silence. | [nextjs 02](../../nextjs-frontend/02-coding-principles.md) |
| 4 | **JSX must have one parent** | Wrap siblings in `<>...</>` or a single container. | [nextjs 11](../../nextjs-frontend/11-ui.md) |
| 5 | **Unexpected token** | Check line in message; look for unclosed `({[` above it. | — |
| 6 | **Duplicate identifier** | Remove or rename duplicate import/const/function. | [backend 01](../../python-fastapi-backend/01-coding-principles.md) |
| 7 | **X not assignable to Y** | Align argument type with function signature. | [nextjs 02](../../nextjs-frontend/02-coding-principles.md) |
| 8 | **import outside a module** | Align ESM/CJS: `"type": "module"` or consistent `require`. | — |
| 9 | **React must be in scope** | Use modern JSX transform or `import React`. | — |
| 10 | **Invalid hook call** | Hooks only at top level of client components — not in conditions. | [nextjs 05](../../nextjs-frontend/05-server-client.md) |

---

## B · Runtime & logic (11–25)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 11 | **Cannot read properties of undefined** | Optional chaining `?.`; guard before access; validate API shape. | [nextjs 07](../../nextjs-frontend/07-data.md) |
| 12 | **Hydration mismatch** (Next.js) | No `window`/`localStorage` during SSR; browser-only code in client leaf + `useEffect` if needed. | [nextjs 05](../../nextjs-frontend/05-server-client.md) |
| 13 | **Infinite re-render** | Fix `useEffect` deps; do not set state that re-triggers same effect. | [nextjs 13](../../nextjs-frontend/13-state.md) |
| 14 | **State not updating** | Functional `setState`; do not read stale state right after set. | [nextjs 13](../../nextjs-frontend/13-state.md) |
| 15 | **Memory leak in useEffect** | Return cleanup for subscriptions, timers, listeners. | [nextjs 05](../../nextjs-frontend/05-server-client.md) |
| 16 | **Maximum call stack** | Add recursion base case or replace with iteration. | [backend 01](../../python-fastapi-backend/01-coding-principles.md) |
| 17 | **Array index out of bounds** | Check `length` / use `.at()`; handle empty lists in UI (four states). | [nextjs 01](../../nextjs-frontend/01-design.md) |
| 18 | **Off-by-one pagination** | Align page index with API (`page` 0 vs 1); document in OpenAPI. | [backend 12](../../python-fastapi-backend/12-api.md) |
| 19 | **Race condition (async)** | `await` sequence; abort stale requests; idempotent writes on server. | [backend 06](../../python-fastapi-backend/06-database.md) |
| 20 | **Stale closure in handlers** | `useRef` for latest value or functional updates. | [nextjs 13](../../nextjs-frontend/13-state.md) |
| 21 | **Date / timezone wrong** | Store UTC in DB; convert for display only. | [backend 06](../../python-fastapi-backend/06-database.md) |
| 22 | **Float precision (money)** | Integer minor units (cents) in API/DB. | [backend 06](../../python-fastapi-backend/06-database.md) |
| 23 | **Encoding in URL/JSON** | `encodeURIComponent`; parameterized SQL — never concat user input. | [backend 15](../../python-fastapi-backend/15-security.md) |
| 24 | **CSS / Tailwind conflicts** | Tokens in [11-ui](../../nextjs-frontend/11-ui.md); avoid fighting library selectors. | [nextjs 01](../../nextjs-frontend/01-design.md) |
| 25 | **No loading / error UI** | Add loading, empty, error, success — mandatory four states. | [nextjs 01](../../nextjs-frontend/01-design.md) |

---

## C · API & data (26–35)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 26 | **CORS error** | Configure CORS on **FastAPI** for browser origin; UI calls API via [09-api-client](../../nextjs-frontend/09-api-client.md). | [backend 10](../../python-fastapi-backend/10-http.md) |
| 27 | **401 Unauthorized** | HttpOnly session cookie or Bearer from server; refresh on FastAPI — not ad-hoc headers in features. | [backend 13](../../python-fastapi-backend/13-identity-security.md), [nextjs 12](../../nextjs-frontend/12-auth.md) |
| 28 | **404 on API route** | Match mounted router path in `http/router.py` + module router prefix. | [backend 10](../../python-fastapi-backend/10-http.md), [09-modules](../../python-fastapi-backend/09-modules.md) |
| 29 | **429 Too Many Requests** | Backoff + cache; rate limits from `Settings`, not hardcoded. | [backend 03](../../python-fastapi-backend/03-config.md), [16](../../python-fastapi-backend/16-performance.md) |
| 30 | **JSON parse error** (HTML body) | Wrong URL or 502/404 page; check Network tab status + response type. | [backend 12](../../python-fastapi-backend/12-api.md) |
| 31 | **DB connection timeout** | Pool size/timeouts in `Settings`; close sessions per request/job. | [backend 03](../../python-fastapi-backend/03-config.md), [06](../../python-fastapi-backend/06-database.md) |
| 32 | **SQL injection** | SQLAlchemy/repository only — **no** string-built SQL from user input. | [backend 06](../../python-fastapi-backend/06-database.md), [15](../../python-fastapi-backend/15-security.md) |
| 33 | **Missing env var** | Add field to `Settings` / `lib/env.ts` + `.env.example`; verify CI/host env. | [backend 03](../../python-fastapi-backend/03-config.md), [nextjs 04](../../nextjs-frontend/04-config.md) |
| 34 | **Webhook signature fail** | Use correct secret per environment in `Settings`; raw body for verify. | [backend extra 09](../../python-fastapi-backend/extra/09-webhooks.md) |
| 35 | **JSON serialization error** | No `Date`/`Decimal`/NaN in response — use DTOs/schemas (Pydantic). | [backend 12](../../python-fastapi-backend/12-api.md) |

**API debug order:** (1) HTTP status → (2) browser Network tab → (3) FastAPI logs with `request_id` ([04-logging](../../python-fastapi-backend/04-logging.md)).

---

## D · Auth & authorization (36–43)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 36 | **Redirect loop on login** | Exclude public routes (login, callback) from auth middleware. | [nextjs 06](../../nextjs-frontend/06-routing.md), [12-auth](../../nextjs-frontend/12-auth.md) |
| 37 | **Session not persisting** | Cookie `HttpOnly`, `Secure` (HTTPS), `SameSite` matching deployment. | [nextjs 12](../../nextjs-frontend/12-auth.md), [backend 13](../../python-fastapi-backend/13-identity-security.md) |
| 38 | **OAuth callback mismatch** | IdP redirect URI must **exactly** match production URL. | [extra SSO](../../python-fastapi-backend/extra/07-sso.md), [nextjs extra 04](../../nextjs-frontend/extra/04-sso.md) |
| 39 | **RLS / tenant blocks all rows** | Explicit tenant filter in repository + tests — do not rely on “magic” defaults. | [extra multi-tenant](../../python-fastapi-backend/extra/01-multi-tenant.md) |
| 40 | **Token in localStorage** | **MUST NOT** — session in HttpOnly cookie; FastAPI owns authz. | [nextjs 12](../../nextjs-frontend/12-auth.md) |
| 41 | **Missing authorization check** | Every protected route: identity + permission in **service**, not UI-only. | [backend 13](../../python-fastapi-backend/13-identity-security.md) |
| 42 | **Password reset token never expires** | Short TTL in `Settings`; one-time use in DB. | [backend 13](../../python-fastapi-backend/13-identity-security.md) |
| 43 | **CSRF on mutations** | SameSite cookies + server-side writes via FastAPI; no secret in `NEXT_PUBLIC_`. | [nextjs 15](../../nextjs-frontend/15-security.md) |

---

## E · Deployment (44–50)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 44 | **Build OK locally, fails in CI** | Pin Node/Python versions; set all env vars; fix path case. | [nextjs 04](../../nextjs-frontend/04-config.md), [backend 03](../../python-fastapi-backend/03-config.md) |
| 45 | **Serverless timeout** | Move heavy work to **worker** queue — not HTTP request. | [backend 11](../../python-fastapi-backend/11-workers.md) |
| 46 | **OOM during build** | Optimize images; trim deps; split apps if needed. | [nextjs 16](../../nextjs-frontend/16-performance.md) |
| 47 | **Mixed content (HTTP on HTTPS)** | All asset/API URLs HTTPS in env. | [nextjs 04](../../nextjs-frontend/04-config.md) |
| 48 | **Static gen error for dynamic page** | Server fetch + dynamic route; no stale static data for user-specific pages. | [nextjs 06](../../nextjs-frontend/06-routing.md), [07](../../nextjs-frontend/07-data.md) |
| 49 | **SSL / DNS not ready** | Wait for propagation; verify cert on host before cutting traffic. | — |
| 50 | **Cold start latency** | Long jobs off HTTP; cache warm paths; pools in `Settings`. | [backend 16](../../python-fastapi-backend/16-performance.md) |

---

## Where to start (by situation)

| You are… | Focus first |
|----------|-------------|
| New to vibe coding | #1–10, #11–15, #26–28 (~80% of first project pain) |
| Backend-heavy | #26–35, #36–41, [backend 15-security](../../python-fastapi-backend/15-security.md) |
| Frontend-heavy | #11–13, #25, [nextjs 01-design](../../nextjs-frontend/01-design.md), [07-data](../../nextjs-frontend/07-data.md) |
| Production incident | #33, #27, #31, #44 + git revert checkpoint ([04 Pitfalls §3](04-pitfalls.md)) |

---

**Next:** [01 Start here](01-start-here.md) · [02 How to prompt](02-how-to-prompt.md) · [03 Review agent code](03-review-agent-code.md) · [04 Pitfalls](04-pitfalls.md) · [05 Errors](05-errors.md) · [06 Glossary](06-glossary.md)
