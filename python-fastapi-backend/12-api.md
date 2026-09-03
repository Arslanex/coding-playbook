# 12 · API contract

WHEN: a public URL, success/error JSON, status code, list page, `error_code`, or correlating a failed request.
LOAD: this file only.
RELATED: 05 (exception classes and the handler) · 09 (module router/schemas) · 10 (shell folders) · 04 (log fields) — open only if the task is also that topic.
SCOPE: what the HTTP **client** sees. Not `http/` file placement (10). Not where `OrderNotFoundError` is declared (05).

The raise site never builds HTTP. The router never catches to invent a body. One success shape. One error shape.

---

## URL

MUST: `/v1/<resources>` — version in the path, not a header. Resource plural, kebab-case (`/v1/orders`, `/v1/stage-assignments`).
ADAPTED (GİRVAK, vidinsight-blog-service): `/v1/<audience>/<resources>` — `/v1/public/posts`, `/v1/admin/posts`, `/v1/ingest/posts`. The reason and the folder consequence are in [09](09-modules.md#package-name); the short version is that the edge proxy decides what is reachable from the internet by path, and one resource here has three callers with three credentials. The rest of this rule holds: plural, kebab-case, no verb as a collection.
MUST NOT: `/api/v1`. MUST NOT: `http/v1/` folder (09, 10).
MUST NOT: a verb as a collection (`/cancel`). Action that is not CRUD = sub-resource `POST /v1/orders/{id}/cancel`.

`/v2` only when the public contract breaks. Adding an optional field, a new endpoint, or a new `error_code` is not a break.

Retiring `/v1` is three steps, not a delete:

1. `/v2` ships. `/v1` keeps working, unchanged.
2. `/v1` responses carry `Deprecation: true` and `Sunset: <HTTP-date>` (the date it stops serving), plus `Link: <…>; rel="successor-version"`. The date comes from `config/` (03), not a literal in a router.
3. After that date `/v1` returns `410 Gone` with `error_code: "API_VERSION_SUNSET"` for at least one more release, then the routes are removed.

MUST: log `LoggerName.API` WARNING with the route when a deprecated version is called, so you can see whether anyone is still on it before step 3.
MUST NOT: delete `/v1` on the day `/v2` ships. MUST NOT: a sunset date with no client actually migrated off.

---

## Success body

MUST: the resource itself. MUST NOT: `{ "data": … }`, `{ "success": true }`, `{ "result": … }`.

```
GET/PATCH  200    OrderResponse
POST       201    OrderResponse + Location: /v1/orders/{id}
POST job   202    { "job_id": "<uuid>" }     # service returned the id (11)
DELETE     204    empty body
GET list   200    { "items": [...], "next_cursor": "<str|null>", "limit": <int> }
```

MUST: `response_model` on every handler that returns a body. A `204` handler declares `status_code=204` and returns `None` instead — a `response_model` with an empty body is a contradiction, and FastAPI will not serialise one.
MUST NOT: return `dict`.
MUST: field names the client sees are declared on the module schema (09). MUST NOT: dump the ORM.

List: every list is paged. `limit` capped in `config/` (default 100). Deterministic `ORDER BY` plus `id` tie-break. `next_cursor` is opaque; `null` = last page. MUST NOT: an unpaged `list[OrderResponse]`.
Until two modules share the envelope, it lives in that module's `schemas.py`. Then `http/pagination.py` (10). MUST NOT: `total` "for convenience" if it requires a second count query the client does not need.

---

## Error body (the only error JSON)

Every 4xx/5xx:

```
{
  "error_code": "ORDER_NOT_FOUND",
  "message": "Sipariş bulunamadı",
  "details": {}
}
```

MUST: all three keys always present. `details` is `{}` when empty — clients MUST NOT special-case a missing key.
MUST: `message` in the product language (user). MUST: `error_code` SCREAMING_SNAKE (machine). Clients branch on `error_code`, never on `message` text.
MUST NOT: a second error shape. MUST NOT: `{"detail": "…"}` from FastAPI leaking through — the handler (05) rewrites it.

`details` — ids and field names the caller already knows (`order_id`, `fields: {path: msg}`). MUST NOT [critical]: SQL, stack, DSN, file paths, another user's ids, tokens.

Who writes this: `http/errors/` only (05). Router raises nothing as JSON. Service raises a typed error.

---

## Status and `error_code`

HTTP status comes from the exception instance (`http_status_code`). Feature `error_code` overrides the parent's generic code.

Parents (05) — do not invent a parallel list:

- 401 `UNAUTHENTICATED` — no/invalid Bearer
- 403 `FORBIDDEN` — authenticated, not allowed (not used for "exists but not yours")
- 404 `NOT_FOUND` / `ORDER_NOT_FOUND` — missing **or** not owned (same code; a 403 would confirm the id)
- 409 `CONFLICT` / `ORDER_NOT_CANCELLABLE` — duplicate or illegal transition
- 422 `VALIDATION_ERROR` — FastAPI body/query schema **or** business `ValidationError`
- 429 `RATE_LIMIT_EXCEEDED` — plus `Retry-After` (and limit headers if rate_limit middleware exists, 10)
- 503 `SERVICE_UNAVAILABLE` — DB/cache/queue/vendor down
- 500 `INTERNAL_ERROR` — unhandled. `message` is generic. MUST NOT: `str(exc)` in the body

201/202/204 as in Success body. 200 for read and in-place update.

MUST: `responses=` on the handler lists the error codes this route can produce (OpenAPI). The error component schema lives in `http/errors/` (05).

---

## How an error leaves the process

```
service raise OrderNotFoundError
  → bubbles (router does not catch)
  → http/errors handler
       status = exc.http_status_code
       body   = {error_code, message, details}
       header X-Request-ID = bound request_id
       log (below)
  → client
```

Unhandled `Exception` → 500 `INTERNAL_ERROR`, generic `message`, stack in the log only.

Workers: same raise, **no** this JSON (05, 11).

---

## How it is traced

Correlation id is `request_id` (04). One value, three places:

1. **Header** — client may send `X-Request-ID`. Middleware accepts it or generates one. MUST: echo `X-Request-ID` on **every** response (2xx and 4xx/5xx). Support asks for this header, not the `message`.
2. **Log context** — every line has `request_id` and `user_id` (null until deps). MUST NOT: copy them into `extra=` (04).
3. **Handler `extra=`** — `error_code`, `status_code`. 4xx: `LoggerName.API` WARNING, no stack. 5xx: `LoggerName.ERROR` ERROR, `exc_info=True`. MUST NOT: log the raw body (passwords).

Query in the log store: `request_id="<the header>"` or `error_code="ORDER_NOT_FOUND"`.

Metrics (10 `metrics.py` when present): count/latency by status class. MUST NOT: a metric named `order_cancelled_total` here — that is `audit` (04) on the service.

`audit` logger: reconstructible writes (create/cancel/money/permission). Not every 404.

---

## Router (thin)

Handler: parse schema → `OrderService(session)` → return schema. Permission via `Depends`, not `if` in the body. Ownership 404 is the service (05).

MUST NOT: `HTTPException`, `JSONResponse`, repository, `commit()`.

Creating `POST`: `Idempotency-Key` when the client retries (10). Replay returns the **first success** body. Failed attempts are not stored as success.

---

## Done

- [ ] Success is the resource (or list envelope / `job_id` / empty); no `data` wrapper
- [ ] Error is always `{error_code, message, details}`; status from the exception
- [ ] Missing and not-owned are the same 404 `error_code`
- [ ] `X-Request-ID` echoed; logs join on that id; 5xx stack is log-only
- [ ] Client can branch on `error_code`; `message` is user copy
- [ ] Router did not build the error JSON
