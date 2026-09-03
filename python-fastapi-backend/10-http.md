# 10 · HTTP

WHEN: middleware, a FastAPI dependency, the error JSON map, the mount list, CORS, pagination envelope, or "does this belong in `http/` or a module?".
LOAD: this file only.
RELATED: 02 (placement) · 13 (JWT, sessions, cache keys) · 05 (the exception types this layer maps) · 12 (what the client sees) — open only if the task is also that topic.
SCOPE: `src/http/` and how `main.py` wires it. Feature routes and HTTP schemas live in `modules/` (09), not here.

`http/` is the transport shell. This process speaks HTTP. It does not know orders.

MUST NOT: `commit()` anywhere in `http/` — not in middleware, not in a dep, not in a router. The service owns the transaction (06).

---

## When code belongs here

Stop at the first yes.

1. A product URL, request/response field, permission on a resource, or `OrderService(…)` factory?
   → **not http** — `modules/<capability>/` (09). Then one include line here.
2. Login, password, refresh, "issue a token" as a product?
   → **not http** — `modules/auth/`. This layer only **reads** a Bearer token.
3. Base exception class or logger pipeline?
   → `shared/` (05, 04). This layer **maps** exceptions to JSON; it does not define `NotFoundError`.
4. Open/close the engine, Redis, queue — process lifetime?
   → `main.py` lifespan. This layer does not own pools.
5. True for (almost) every request, and the file name has no product noun?
   → `http/` then the matching child below.

MUST NOT: `src/api/` as a second tree. MUST NOT: `http/v1/` — version is `APIRouter(prefix="/v1/…")` on the **module** router (09).

---

## Day-one files

```
src/http/
  router.py         # include list only
  middleware/
    request_id.py   # bind/reset log context (04)
  deps.py           # get_session, get_current_user. Folder later (01).
  errors/           # exception → {error_code, message, details} (05)
```

`main.py` — app factory. Adds middleware **in order**, registers `http/errors` handlers, includes `http/router.py`, lifespan opens/closes infra (06, 08) and `configure_logging()` (04). MUST NOT: a route handler in `main.py`. MUST NOT: `include_router(orders.router)` in `main.py` — that is the mount list.

Tests: `tests/http/` mirrors this tree. Product route tests live under `tests/modules/`.

---

## Relation to other layers

```
main.py
  lifespan: config + infra pools + logging
  middleware stack (http/middleware)
  exception handlers (http/errors)
  include http/router.py
    include modules/<capability>/router.py     # prefix="/v1/…" lives there
      Depends: http/deps  (session, user)
      schemas + OrderService(session)          # module
        → infra/db  / other services           # module, not http
        → commit                               # service, not http

workers/jobs/  →  modules/*/service.py         # skip http entirely (11)
```

`http/` → may import: `shared/errors` (to catch parents), `shared/logging` (bind context, `LoggerName.API`), `infra/db/session.py` (open/close scope only), `config/` (CORS origins, JWT secret, rate limits).

`http/` → MUST NOT import: `infra/db/models`, `infra/db/repositories`, `infra/queue`, `infra/storage`, any `modules/*/service.py`, any `modules/*/schemas.py`.

`infra/cache` is the one exception, and only for these four transport concerns: rate-limit counters, idempotency replay, the access-token `jti` denylist, and a session/identity lookup in `deps` (08, 13). Anything else in Redis is a module's decision — `http/` MUST NOT pick a cache key that names a product noun.

Exception: `http/router.py` imports **router objects** from modules (`modules.orders.router`). That is the mount list. It does not import services.

Modules → import `http/deps` (`get_session`, `get_current_user`). MUST NOT: import `http/middleware` or `http/errors` (those are registered in `main.py`). MUST NOT: `http/` import a module service.

Infra → MUST NOT import `http/`.

Workers → MUST NOT import `http/`. Same session scope as 06, opened in the job, not via `get_session`.

---

## `router.py` — mount list

Is: the include list. One `include_router` per module router. Optional: a liveness route with **no** service (`GET /health` → `{"status": "ok"}`).

MUST NOT: path handlers for orders/auth. MUST NOT: `prefix="/v1"` wrapping all includes — each module sets its own `/v1/<resource>` (09). MUST NOT: grow this file into a second API.

A new public URL: add the route on the **module** router, then one include line here if the package is new.

---

## `middleware/`

Is: runs on (almost) every request. No product noun in the filename.

Day one: `request_id.py` — generate or accept `X-Request-ID`, bind log context, echo the same value on the response (12), reset in `finally` (04). MUST NOT: log the full body (passwords).

Add a file when the trigger is true:

- `cors.py` — browser clients. Origins from `config/`.
- `rate_limit.py` — per-IP or per-user cap. Counter in `infra/cache/` if it must be shared across pods; in-memory only if a single process is acceptable. MUST NOT: "user may place 3 orders" — that is a module rule.
- `metrics.py` — request count/latency histograms. MUST NOT: `order_cancelled_total` here (module/`audit` logger).
- Idempotency — only when clients send `Idempotency-Key` on POST. Store the HTTP outcome in `infra/cache/`. MUST NOT: encode cancel policy. Replay is transport.

Order in `main.py` (outside → inside):

1. `request_id` (context exists before anything logs)
2. CORS — unless an edge fronts every browser call and owns the origin list ([Extra 02](extra/02-microservices.md)). One place, never two.

ADAPTED (GİRVAK, vidinsight-blog-service): CORS is **outermost**, above `request_id`. A response built by a middleware never reaches the layers outside it, so with CORS inside, the rate limiter's own 429 carried no `Access-Control-Allow-Origin` and a browser reported a bare network error instead of the status it was sent. `request_id` still binds before anything that logs — CORS logs nothing — so the reason behind rule 1 holds. Note this only matters while `CORS_ALLOWED_ORIGINS` is non-empty; in this product it is empty by default, because no browser calls the API (the panel goes through the site's same-origin proxy).
3. metrics
4. rate_limit
5. idempotency (if present)
6. route → deps → module

MUST: reset contextvars in `finally`. MUST NOT: call a module service. Logger name: `api` (04).

---

## `deps.py` — session + current user

Is: who is calling, and which DB session. Not named `auth`.

`get_session` — enter `infra/db/session.py` scope, yield `AsyncSession`, on exit rollback-if-raised then close (06). MUST NOT: nest a second session.

`get_current_user` — Bearer JWT (secret from `config/`) → `user_id` (and a small `CurrentUser` if more claims are required). Bind `user_id` on log context (04). MUST NOT: login, password, refresh, signup — `modules/auth/`. MUST NOT: `OrderService`. MUST NOT: `HTTPException` with a custom body — raise `AuthenticationError` / `AuthorizationError` (05) and let `http/errors/` map.

Day one: claims from the token are enough. MUST NOT: `UserRepository` "just in case". If a later requirement is "reject revoked sessions", the lookup is still this dep (identity), not `AuthService.login`. **The dep does that read itself, through `infra/`.** MUST NOT: reach a module service to do it.

**Still no product sentence** — that is the test, and it is the only one this page applies. "Is this token revoked, is this account still active" names no product noun; "who may be invited, what a password must look like, when a refresh rotates" does, and stays in `modules/auth/`.

Which store backs that read is a **durability** question, answered in [13](13-identity-security.md). It changes nothing here. This line used to name one store, and that is precisely what broke: a product that made its revocation list durable — as 13 tells it to — then had no sanctioned way to read it from `http/`, because the only door left was a module service this page forbids. A layering rule that answers with a storage location stops answering the moment the storage changes.

Public routes omit `get_current_user`. They may still take `get_session` if they write.

Module factory stays on the module router: `OrderService(session)`. `deps.py` does not construct services. `modules/*/deps.py` only if that factory is fat (09).

Split `deps/` into files only when 01 fires (session vs identity as two reasons to change). MUST NOT: `http/deps/auth.py` as a name — identity, not the auth product.

---

## `errors/` — the only HTTP map

Is: exception → status + `{error_code, message, details}`. Registered from `main.py`. Four handlers (05). What the client sees: 12. Logger: `api` for expected 4xx, `error` for unhandled 5xx.

MUST NOT: a second JSON shape. MUST NOT: a module `except OrderNotFoundError` to build a body. Exception classes belong to their module or `shared/errors` — 05 owns that rule.

OpenAPI error component lives here, next to the handlers, until 01 split (05).

---

## Extra files

`pagination.py` — two or more modules return a list page (`items` + `next_cursor` / `limit`). Until then the first list declares the envelope in that module's `schemas.py`. MUST NOT: `shared/pagination.py`.

MUST NOT: `http/schemas/` (`OrderResponse`), `http/orders.py`, `http/services/`, `http/utils.py`.

---

## Done

- [ ] File name has no product noun; product URL lives in a module
- [ ] `router.py` is still an include list (plus optional `/health` with no service)
- [ ] `deps.py` is session + current user; no `commit()`; no `*Service`
- [ ] `http/` does not import module services, models, or repositories
- [ ] Workers still skip this layer
- [ ] Error body still `{error_code, message, details}` (05)
- [ ] `request_id` bound in middleware, `user_id` bound in deps (04)
