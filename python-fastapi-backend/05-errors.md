# 05 · Errors

WHEN: adding an exception class, raising from a service, mapping to HTTP, or changing the error JSON.
LOAD: this file only.
RELATED: 12 (what the client sees) · 09 (module `errors.py` trigger) · 10 (handler registration) · 02 (placement) — open only if the task is also that topic.
SCOPE: `shared/errors/`, `http/errors/`, and feature exceptions inside `modules/`.

One hierarchy. One JSON shape. Three homes. The raise site never builds HTTP.

---

## Three homes

```
shared/errors/     kernel types. no product noun.
modules/<name>/    feature types that subclass the kernel. raise here.
http/errors/       exception → status + JSON. the only HTTP map.
```

MUST: every application error subclasses `AppBaseError`.
MUST NOT: `HTTPException` or `JSONResponse` in a module or in infra.
MUST NOT: a second error JSON shape.

Workers use the same types. They do not map to HTTP. The runner logs with `LoggerName.WORKER`, level ERROR, `exc_info=True` (04) and fails the job (11). MUST NOT: `LoggerName.ERROR` in a worker — that name is the HTTP 5xx handler only.

---

## Folder vs file

`shared/` and `http/` already have an `errors/` **folder** in the tree (02). That folder is the package. Inside it, start with **files**. Do not nest another folder until 01 split rules fire (two reasons to change, or >300 lines).

`shared/errors/`
- Holds: `AppBaseError` and the status-class parents (`NotFoundError`, `AuthenticationError`, `AuthorizationError`, `ValidationError`, `ConflictError`, `RateLimitError`, `ServiceUnavailableError`).
- One file is enough for that list. Split a file only when 01 says so.
- MUST NOT: `OrderNotFoundError` here. MUST NOT: `orders.py` inside `shared/errors/`.
- MUST NOT: the HTTP handler here. Mapping is transport → `http/errors/`.

`http/errors/`
- Holds: the global handlers and the OpenAPI error body schema.
- One file is enough until handlers and the OpenAPI map have different reasons to change — then two files in the same folder, still no subfolder.
- MUST NOT: new exception classes here.

`modules/<capability>/`
- Holds: feature exceptions (`OrderNotFoundError`).
- 1–2 classes: next to the service in the same file.
- 3+ classes: one `errors` file in that module. MUST NOT: `modules/orders/errors/` as a folder.

---

## Kernel types (`shared/errors/`)

Each parent is a **status class**. It sets `http_status_code` and a generic `error_code`. Feature types override `error_code` and the user `message`.

- `AppBaseError` — 500 / `INTERNAL_ERROR`. Fields: `message`, `details` (dict, default `{}`).
- `NotFoundError` — 404 / `NOT_FOUND`
- `AuthenticationError` — 401 / `UNAUTHENTICATED`
- `AuthorizationError` — 403 / `FORBIDDEN`
- `ValidationError` — 422 / `VALIDATION_ERROR` (business rule, not JSON schema — that is FastAPI's `RequestValidationError`, mapped in `http/errors/`)
- `ConflictError` — 409 / `CONFLICT`
- `RateLimitError` — 429 / `RATE_LIMIT_EXCEEDED`
- `ServiceUnavailableError` — 503 / `SERVICE_UNAVAILABLE` (dependency down: DB, cache, queue)

MUST: add a new **parent** only when a new HTTP status is needed for many features. A one-off status is still a feature class with `http_status_code` set on that class, subclassing the closest parent.

**Name collision.** Our `ValidationError` and Pydantic's `ValidationError` are different types with the same name, and a FastAPI codebase imports both. A bare `except ValidationError` that caught the wrong one is a silent bug — the handler never fires and the client gets a 500.

MUST: import ours plainly and alias the framework's at the import site.

```python
from shared.errors import ValidationError                  # ours: business rule, 422
from pydantic import ValidationError as PydanticValidationError
from fastapi.exceptions import RequestValidationError      # what the handler registers on
```

MUST NOT: `from pydantic import *`, or a local `ValidationError` shadowing either. MUST NOT: register a handler on Pydantic's class expecting FastAPI request bodies — FastAPI raises `RequestValidationError` for those (the handler below).

MUST NOT: put `http_status_code` logic in the handler per class name. The handler reads the instance.

---

## Feature types (`modules/<capability>/`)

```python
class OrderNotFoundError(NotFoundError):
    error_code = "ORDER_NOT_FOUND"

    def __init__(self) -> None:
        super().__init__("Sipariş bulunamadı")
```

MUST: `error_code` SCREAMING_SNAKE, unique, prefixed by the noun (`ORDER_…`).
MUST: `message` in the product language (user-facing; 01). Docstring English (agent).
MUST [critical]: missing and not-owned use the **same** type and 404. A 403 would confirm the id exists.
MUST: `details` only ids and field names the client may see. MUST NOT [critical]: SQL, stack traces, secrets, another user's ids, file paths.

Raise in the service (or a helper it calls). The router does not catch and wrap.

Infra may raise `ServiceUnavailableError` when a dependency is down. Infra MUST NOT raise feature types (`OrderNotFoundError`).

---

## HTTP map (`http/errors/`)

Register on the app from `main.py`. Four handlers, same body builder:

```
{ "error_code": str, "message": str, "details": object }
```

`details` is always present (empty object if none). Clients must not special-case a missing key.

1. `AppBaseError` → `exc.http_status_code`, `exc.error_code`, `exc.message`, `exc.details`.
   - status < 500: log WARNING, no stack (expected business outcome).
   - status >= 500: log ERROR, `exc_info=True` (04: `LoggerName.ERROR`).
2. `RequestValidationError` → 422 / `VALIDATION_ERROR`. `details.fields` = `{path: msg}`. Log field names, not the raw body (it may contain a password).
3. Starlette `HTTPException` (framework 404/405) → same JSON shape. Map 404 → `NOT_FOUND`, 405 → `METHOD_NOT_ALLOWED`.
4. bare `Exception` → 500 / `INTERNAL_ERROR`, generic user message. MUST NOT: put `str(exc)` in the body. Full exception goes to the log only.

MUST: unhandled failures use a generic user `message`. Specific `message` values live on feature exceptions that were raised on purpose.

---

## Call chain

```
modules (raise OrderNotFoundError)
  → http/errors (JSON + status)
  → client

workers/jobs (same raise)          # 11: runner classifies retry vs DLQ
  → no HTTP map
  → log + fail the job
```

MUST NOT: a router `except OrderNotFoundError` to return a body.
MUST NOT: a service `return None` for not-found when the caller must handle absence — raise.

---

## Done

- [ ] New type subclasses a kernel parent, not `Exception`
- [ ] Feature type lives in the module, not in `shared/errors/`
- [ ] Handler lives in `http/errors/`, not in `shared/`
- [ ] `shared/errors/` and `http/errors/` stayed folders of files, no extra nesting
- [ ] Body is still `{error_code, message, details}`
- [ ] 4xx not logged as ERROR with stack
- [ ] Secrets / SQL / stack not in `details` or in the client `message`
