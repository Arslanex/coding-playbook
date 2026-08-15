# 09 · API client

WHEN: calling FastAPI, parsing JSON, setting headers, or handling `{error_code, message, details}`.
LOAD: this file. Backend contract: [python-fastapi-backend/12-api.md](../python-fastapi-backend/12-api.md) when the shape is in doubt.
RELATED: 04 (base URL) · 07 (reads) · 08 (writes) · 12 (cookies) — open only if the task is also that topic.
SCOPE: `frontend/lib/api.ts` plus per-feature functions. Not a second REST framework.

One wrapper. Features call typed functions, not raw `fetch` sprinkled in JSX.

---

## Wrapper (`lib/api.ts`)

MUST:

- Prefix `API_BASE_URL` (04). Paths start `/v1/…` — never `/api/v1` on FastAPI. **One** base URL even when the backend is several services: they sit behind one origin, and which service answers is not this app's concern (backend Extra 02).
- Forward cookies on the server (`credentials: "include"` in the browser).
- Send and echo `X-Request-ID` (generate if missing).
- Timeouts from `lib/env.ts`.
- JSON. Empty 204: no body.

MUST NOT: axios as a second stack unless the project already has it — then still one wrapper. MUST NOT: a new client per feature.

---

## Success and error

Success: the resource or `{ items, next_cursor, limit }` or `{ job_id }` (202). MUST NOT: unwrap `.data` that the API does not send.

Error: always `{ error_code, message, details }`. Throw a typed `ApiError` with those fields. Reporting happens **here**, once, not in every caller — which errors are worth reporting: 06. UI branches on `error_code`, shows `message` (01). MUST NOT: branch on HTTP status alone when `error_code` exists. MUST NOT: show `details` stacks.

401 → sign-in flow (12). 404 → not-found UI (same for missing and not-owned). 429 → `Retry-After` if present.

---

## Feature functions

`features/orders/api.ts`: `getOrder`, `listOrders`, `cancelOrder`. Zod parse. MUST NOT: `features/orders/fetch.ts` that returns `any`.

---

## Done

- [ ] One wrapper; `/v1`; cookies; request id
- [ ] Zod at the boundary; `ApiError` carries `error_code`
- [ ] No `.data` wrapper the API does not use
