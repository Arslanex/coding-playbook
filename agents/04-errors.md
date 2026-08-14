# 04 · Error routing (50 patterns)

WHEN: the user pasted a build log, runtime stack trace, HTTP failure, auth bug, or deploy error — or said "it broke" **with** a specific message.
LOAD: this file and [02-turn.md](02-turn.md). Then the stack file named in the matched block's `LOAD:` line (+ that file's `LOAD:` siblings only).
RELATED: [03-anti-patterns.md](03-anti-patterns.md) if the fix tempts a large rewrite.
SCOPE: match error text → WHERE to edit → HOW to fix. Human checklist: [VIBE-CODING-ERRORS.md](../VIBE-CODING-ERRORS.md).

MUST: read the **verbatim** error string first.
MUST: use the **first** block below whose WHEN line matches.
MUST: change only what fixes this error.
MUST NOT: fix without error text — ask user or read terminal.
MUST NOT: refactor unrelated code.

WHEN: API failure and status code is visible.
HOW: diagnose in order — (1) HTTP status (2) response body is JSON or HTML (3) server log with `request_id` ([backend 04-logging](../python-fastapi-backend/04-logging.md)).

WHEN: multiple unrelated errors in one message.
HOW: fix in order — build blockers → env/auth (33, 40, 27) → security (32, 41) → runtime UX (11–25) → deploy (44–50).
MUST NOT: bundle unrelated fixes in one turn unless user listed them all.

---

## A · Build and compilation

### A01 · Missing npm/pip package

WHEN: log contains `Module not found` or `Cannot find module` for a **package name** (not a relative `./` or `../` path).
WHERE: `package.json`, `pyproject.toml`, lockfile.
HOW: declare dependency; pin version; install.
LOAD: none unless install layout is unclear.
MUST NOT: shim the import locally instead of declaring the dependency.

### A02 · Wrong relative import path

WHEN: log contains `Cannot find module './` or `Cannot find module '../`.
WHERE: the importing file and the target file — check path and **filename case** (Linux CI is case-sensitive).
HOW: fix import to match actual file location.
LOAD: [nextjs 03-file-structure](../nextjs-frontend/03-file-structure.md).

### A03 · TypeScript property missing

WHEN: log contains `Property '…' does not exist` or `TS2339`.
WHERE: the property access site and the type/API definition.
HOW: align name and shape with schema or API; fix upstream type — do not `as any` to silence.
LOAD: [nextjs 02-coding-principles](../nextjs-frontend/02-coding-principles.md).

### A04 · JSX needs one parent

WHEN: log contains `JSX expressions must have one parent element`.
WHERE: the component `return`.
HOW: wrap siblings in `<>…</>` or one container.
LOAD: [nextjs 11-ui](../nextjs-frontend/11-ui.md).

### A05 · Unexpected token (syntax)

WHEN: log contains `Unexpected token`.
WHERE: line cited in message; scan upward for unclosed `(`, `{`, `[`, quotes.
HOW: fix syntax at source.
LOAD: none.

### A06 · Duplicate identifier

WHEN: log contains `Duplicate identifier`.
WHERE: imports and declarations in the cited file.
HOW: remove or rename the duplicate.
LOAD: [backend 01-coding-principles](../python-fastapi-backend/01-coding-principles.md).

### A07 · Type not assignable

WHEN: log contains `is not assignable to type`.
WHERE: call site and function/param definition.
HOW: match argument type to signature.
LOAD: [nextjs 02-coding-principles](../nextjs-frontend/02-coding-principles.md).

### A08 · ESM vs CommonJS

WHEN: log contains `Cannot use import statement outside a module`.
WHERE: `package.json` `"type"`, import/require usage.
HOW: align module system across config and files.
LOAD: none.

### A09 · React not in scope

WHEN: log contains `'React' must be in scope when using JSX`.
WHERE: component file or JSX runtime config.
HOW: add `import React from 'react'` or enable automatic JSX runtime.
LOAD: none.

### A10 · Invalid hook call

WHEN: log contains `Invalid hook call`.
WHERE: component using hooks — must be `"use client"` leaf; hooks at top level only.
HOW: move hooks out of conditions/nested functions; ensure not calling hooks from non-components.
LOAD: [nextjs 05-server-client](../nextjs-frontend/05-server-client.md).

---

## B · Runtime and logic

### B11 · Undefined/null property access

WHEN: log contains `Cannot read properties of undefined` or `null is not an object`.
WHERE: property access before data is loaded.
HOW: guard with `?.`; validate API payload shape; render loading/empty states first.
LOAD: [nextjs 07-data](../nextjs-frontend/07-data.md).

### B12 · Hydration mismatch

WHEN: log contains `Hydration failed` or `hydration mismatch`.
WHERE: component rendering different output on server vs client.
HOW: remove `window`, `localStorage`, random IDs from SSR; browser-only code in client leaf + `useEffect` if needed.
LOAD: [nextjs 05-server-client](../nextjs-frontend/05-server-client.md).

### B13 · Infinite re-render

WHEN: log contains `Too many re-renders` or effect loop suspected.
WHERE: `useEffect` that sets state also listed in its dependency array.
HOW: fix deps; do not set state that immediately retriggers the same effect.
LOAD: [nextjs 13-state](../nextjs-frontend/13-state.md).

### B14 · Stale state after setState

WHEN: UI reads old value immediately after `setState`.
WHERE: the handler reading state synchronously after set.
HOW: use functional updater `setX(prev => …)`; do not assume new value in same tick.
LOAD: [nextjs 13-state](../nextjs-frontend/13-state.md).

### B15 · useEffect leak

WHEN: warning about missing cleanup or subscription leak.
WHERE: `useEffect` with listeners, intervals, subscriptions.
HOW: return cleanup function from the effect.
LOAD: [nextjs 05-server-client](../nextjs-frontend/05-server-client.md).

### B16 · Stack overflow / infinite recursion

WHEN: log contains `Maximum call stack size exceeded`.
WHERE: recursive function without base case.
HOW: add termination condition or replace with iteration.
LOAD: [backend 01-coding-principles](../python-fastapi-backend/01-coding-principles.md).

### B17 · Empty array / index error

WHEN: index access fails or list assumed non-empty.
WHERE: array access and list render path.
HOW: check `length`; render empty state from four-state pattern.
LOAD: [nextjs 01-design](../nextjs-frontend/01-design.md).

### B18 · Pagination off-by-one

WHEN: wrong page of results or skipped rows.
WHERE: client page param and API contract.
HOW: align 0-based vs 1-based index with backend; document in API.
LOAD: [backend 12-api](../python-fastapi-backend/12-api.md).

### B19 · Async race condition

WHEN: intermittent wrong data after parallel requests.
WHERE: overlapping fetches or writes to same resource.
HOW: `await` in order; abort stale fetch; idempotent server writes.
LOAD: [backend 06-database](../python-fastapi-backend/06-database.md).

### B20 · Stale closure in handler

WHEN: handler uses old state though UI updated.
WHERE: event handler closing over stale render.
HOW: `useRef` for latest value or functional state updates.
LOAD: [nextjs 13-state](../nextjs-frontend/13-state.md).

### B21 · Wrong timezone on dates

WHEN: displayed time wrong across zones.
WHERE: DB storage and UI formatting.
HOW: store UTC in DB; convert to local only at display.
LOAD: [backend 06-database](../python-fastapi-backend/06-database.md).

### B22 · Money float error

WHEN: `0.1 + 0.2` style rounding in prices.
WHERE: amount fields in API/DB.
HOW: integer minor units (cents) end-to-end.
LOAD: [backend 06-database](../python-fastapi-backend/06-database.md).

### B23 · URL or encoding corruption

WHEN: garbled query string or broken special characters.
WHERE: URL construction and SQL/query layer.
HOW: `encodeURIComponent` for URLs; bound parameters for SQL — never concat user input.
LOAD: [backend 15-security](../python-fastapi-backend/15-security.md).

### B24 · CSS / Tailwind conflict

WHEN: styles wrong or overridden unexpectedly.
WHERE: competing class names and library CSS.
HOW: use design tokens from `ui/`; avoid selector wars.
LOAD: [nextjs 01-design](../nextjs-frontend/01-design.md), [11-ui](../nextjs-frontend/11-ui.md).

### B25 · No loading or error UI

WHEN: blank screen after fetch or silent failure.
WHERE: data-fetching component.
HOW: implement loading, empty, error, success — all four states.
LOAD: [nextjs 01-design](../nextjs-frontend/01-design.md).
MUST NOT: add `useEffect` GET when server fetch fits ([nextjs 07-data](../nextjs-frontend/07-data.md)).

---

## C · API and data

### C26 · CORS blocked

WHEN: browser console shows `CORS`, `Access-Control-Allow-Origin`, or blocked cross-origin request.
WHERE: FastAPI HTTP middleware/CORS config — not Next.js as API owner.
HOW: allow browser origin on API; UI calls API per fetch wrapper.
LOAD: [backend 10-http](../python-fastapi-backend/10-http.md), [nextjs 09-api-client](../nextjs-frontend/09-api-client.md).

### C27 · HTTP 401 Unauthorized

WHEN: response status is **401**.
WHERE: cookie/Bearer on server; refresh path on FastAPI — not ad-hoc headers in feature files.
HOW: fix session issuance and attach credentials on server-side fetch or cookie flow.
LOAD: [backend 13-identity-security](../python-fastapi-backend/13-identity-security.md), [nextjs 12-auth](../nextjs-frontend/12-auth.md).

### C28 · HTTP 404 on API route

WHEN: response status is **404** on an API URL the UI calls.
WHERE: `http/router.py` mount list and module router prefix.
HOW: align path with mounted router and handler.
LOAD: [backend 10-http](../python-fastapi-backend/10-http.md), [09-modules](../python-fastapi-backend/09-modules.md).

### C29 · HTTP 429 Too Many Requests

WHEN: response status is **429**.
WHERE: client retry logic and server rate limits.
HOW: exponential backoff; store limits in `Settings` — not literals.
LOAD: [backend 03-config](../python-fastapi-backend/03-config.md), [16-performance](../python-fastapi-backend/16-performance.md).

### C30 · JSON.parse on HTML

WHEN: `JSON.parse` fails or unexpected token `<` in response body.
WHERE: client fetch URL and server route.
HOW: fix endpoint URL; handle non-JSON error responses explicitly.
LOAD: [backend 12-api](../python-fastapi-backend/12-api.md).

### C31 · Database connection timeout

WHEN: log contains `connection timeout`, pool exhausted, too many connections.
WHERE: session lifecycle, pool settings.
HOW: set pool size/timeouts in `Settings`; close sessions per request/job.
LOAD: [backend 03-config](../python-fastapi-backend/03-config.md), [06-database](../python-fastapi-backend/06-database.md).

### C32 · SQL injection risk

WHEN: raw SQL built with user input or string concatenation in queries.
WHERE: repository layer.
HOW: SQLAlchemy/repository with bound parameters only.
LOAD: [backend 06-database](../python-fastapi-backend/06-database.md), [15-security](../python-fastapi-backend/15-security.md).

### C33 · Missing environment variable

WHEN: log contains `Environment variable … not set`, missing env at startup, or CI/host missing key.
WHERE: `Settings` / `lib/env.ts` and `.env.example`; CI/host dashboard.
HOW: add typed field; list name in `.env.example`; set in deployment.
LOAD: [backend 03-config](../python-fastapi-backend/03-config.md), [nextjs 04-config](../nextjs-frontend/04-config.md).

### C34 · Webhook signature invalid

WHEN: Stripe or other webhook returns signature verification failure.
WHERE: webhook handler and `Settings` secret for that environment.
HOW: use correct prod vs test secret; verify against raw request body.
LOAD: [backend extra 09-webhooks](../python-fastapi-backend/extra/09-webhooks.md).

### C35 · Not JSON serializable

WHEN: log contains `Object of type … is not JSON serializable` or similar on API response.
WHERE: response construction — dict with Decimal, datetime, NaN.
HOW: Pydantic response models; serialize before return.
LOAD: [backend 12-api](../python-fastapi-backend/12-api.md).
MUST NOT: put DB credentials or API secrets in frontend env ([nextjs 04-config](../nextjs-frontend/04-config.md)).

---

## D · Auth and authorization

### D36 · Login redirect loop

WHEN: browser bounces endlessly on `/login` or auth routes.
WHERE: auth middleware and public route list.
HOW: exclude login and OAuth callback from auth redirect.
LOAD: [nextjs 06-routing](../nextjs-frontend/06-routing.md), [12-auth](../nextjs-frontend/12-auth.md).

### D37 · Session lost on refresh

WHEN: user logged out after page reload though they just signed in.
WHERE: `Set-Cookie` flags on session cookie.
HOW: `HttpOnly`, `Secure` on HTTPS, `SameSite` matching deployment.
LOAD: [nextjs 12-auth](../nextjs-frontend/12-auth.md), [backend 13-identity-security](../python-fastapi-backend/13-identity-security.md).

### D38 · OAuth redirect_uri_mismatch

WHEN: IdP error contains `redirect_uri_mismatch`.
WHERE: IdP app settings and deployed callback URL.
HOW: make redirect URI **exactly** match production URL.
LOAD: [backend extra 07-sso](../python-fastapi-backend/extra/07-sso.md), [nextjs extra 04-sso](../nextjs-frontend/extra/04-sso.md).

### D39 · Tenant filter blocks all rows

WHEN: queries return empty for all users or tenant isolation too aggressive.
WHERE: repository tenant filter and RLS-style checks.
HOW: explicit tenant predicate; tests for cross-tenant denial.
LOAD: [backend extra 01-multi-tenant](../python-fastapi-backend/extra/01-multi-tenant.md).

### D40 · Token in localStorage

WHEN: code stores JWT or session token in `localStorage` or `sessionStorage`.
WHERE: auth client code in frontend.
HOW: HttpOnly cookie session; authz on FastAPI.
LOAD: [nextjs 12-auth](../nextjs-frontend/12-auth.md).
MUST NOT: leave token in browser storage.

### D41 · Missing authorization in service

WHEN: user can access another user's resource by ID tampering.
WHERE: service method for protected operation — not UI-only hide.
HOW: check identity and permission at start of every protected service call.
LOAD: [backend 13-identity-security](../python-fastapi-backend/13-identity-security.md).

### D42 · Password reset never expires

WHEN: reset token valid indefinitely or reusable.
WHERE: reset token storage and validation.
HOW: short TTL in `Settings`; one-time use in DB.
LOAD: [backend 13-identity-security](../python-fastapi-backend/13-identity-security.md).

### D43 · CSRF on mutation

WHEN: cross-site form post succeeds without protection.
WHERE: cookie session + mutation routes.
HOW: SameSite cookies; writes via FastAPI; no secrets in `NEXT_PUBLIC_*`.
LOAD: [nextjs 15-security](../nextjs-frontend/15-security.md).

---

## E · Deployment

### E44 · CI build fails, local passes

WHEN: pipeline fails while local build succeeds.
WHERE: CI Node/Python version, env vars, import paths.
HOW: pin runtime versions; set all env in CI; fix filename case for Linux.
LOAD: [backend 03-config](../python-fastapi-backend/03-config.md), [nextjs 04-config](../nextjs-frontend/04-config.md).

### E45 · Serverless / HTTP timeout

WHEN: `504`, function timeout, or platform max duration exceeded on long work.
WHERE: HTTP handler doing heavy work.
HOW: enqueue worker job; return `202` — do not run long work in request.
LOAD: [backend 11-workers](../python-fastapi-backend/11-workers.md).

### E46 · Build out of memory

WHEN: `JavaScript heap out of memory` or OOM during build.
WHERE: large deps and unoptimized assets.
HOW: trim dependencies; optimize images; split apps if needed.
LOAD: [nextjs 16-performance](../nextjs-frontend/16-performance.md).

### E47 · Mixed content

WHEN: browser blocks HTTP resource on HTTPS page.
WHERE: env URLs for API and assets.
HOW: all external URLs HTTPS in env.
LOAD: [nextjs 04-config](../nextjs-frontend/04-config.md).

### E48 · Static generation on dynamic page

WHEN: log contains `Dynamic server usage` or SSG error for user-specific data.
WHERE: Next route that needs runtime user data.
HOW: server fetch; mark route dynamic where required.
LOAD: [nextjs 06-routing](../nextjs-frontend/06-routing.md), [07-data](../nextjs-frontend/07-data.md).

### E49 · SSL or DNS not ready

WHEN: certificate error or DNS not propagated.
WHERE: hosting/DNS — not application code hack.
HOW: wait for propagation; verify cert on platform before cutover.
LOAD: none.

### E50 · Cold start latency

WHEN: first request after idle is very slow.
WHERE: long init in HTTP path or unbounded pool on wake.
HOW: move long work off HTTP; tune pools and caps in `Settings`.
LOAD: [backend 16-performance](../python-fastapi-backend/16-performance.md).

---

## Done

MUST: before stopping, confirm:

- Matched block id (A01–E50) stated in reply
- LOAD set was one primary stack file (+ required siblings)
- Verbatim error is gone or user told what blocks fix (env/host)
- No unrelated refactor
