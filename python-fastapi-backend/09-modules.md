# 09 · Modules

WHEN: a product capability, a file inside one, a shape (schema vs dto), a helper, or code two modules both need.
LOAD: this file only.
RELATED: 02 (placement) · 01 (when to split a file) · 12 (public JSON/status) · 06 (where a join goes) — open only if the task is also that topic.
SCOPE: `src/modules/<capability>/`. Model and repository are named here so they are not put in the module; they live in `infra/db/` (06).

A module is one product noun and everything that **means**. It is not a layer (`service`, `api`).

---

## Package name

English, snake_case, the noun: `orders`, `auth`, `billing`.

MUST: folder = resource after the version (`/v1/orders` → `orders`).
MUST NOT: `v1`, `api`, `handlers`, `core`, `orders_service`, a verb (`cancel`).
ADAPTED (GİRVAK, vidinsight-blog-service): this API segments its URLs by **audience**, not by resource — `/v1/public/posts`, `/v1/admin/posts`, `/v1/ingest/posts` — because the three surfaces have different credentials *and* different network exposure, and the edge proxy enforces the difference by path (only `/v1/ingest/*` is published; the site reaches the other two privately). Resource-first URLs would leave nginx nothing to match on. So the segment after the version is an audience, and **the folder still follows the noun, not that segment**: `modules/posts/` owns all three. The audience becomes a router file inside it — see the *Second router file* adaptation below. `modules/public/`, `modules/admin/` and `modules/ingest/` all existed here first and each one collected exactly the coupling this page forbids: they held HTTP for nouns they did not own, so their `schemas.py` became the file every other module imported, and `categories/` was left as a lone `service.py` with its two routes and its response shape living in two other modules. `ingest` was additionally a verb. Pinned by `tests/test_module_boundaries.py`.

New folder only if **deleting it removes a user-visible ability**. Same noun, extra behavior → file in the existing package (01), not a new folder.

`/v1` is the public HTTP major version. It is a prefix string, not a directory.

```
modules/orders/router.py  →  APIRouter(prefix="/v1/orders")
http/                     →  include that router (one line)
```

MUST NOT: `http/v1/`, `src/api/v1/`, `modules/v1/`. MUST NOT: `/v2` until the public contract breaks — still a prefix, still not a folder.
Package, `OrderService`, `orders` table: unversioned. Only the path and `schemas.py` follow `v1`.

---

## Used by more than one module

"Two callers" is not a path. Stop at the first yes.

1. Table, SQL, or vendor client (vendor gone → no product sentence left)?
   → `infra/` (06, 08). Many services may **read** the same repo. **Write** = one owning service.
2. Base exception or logger pipeline (exists with zero features)?
   → `shared/errors/` or `shared/logging/` only. MUST NOT: `shared/money.py`, `shared/slug.py`.
3. Every request as transport (session, current user, error JSON, request id)?
   → `http/`.
4. A product rule (rounding, slug, cancel)?
   → **one** module owns the helper. Others call that **service** (DTO if the shape must cross). MUST NOT: import the helper. MUST NOT: copy it.
5. No owner, and deleting a new folder would remove an ability?
   → new `modules/<capability>/`. Others call **its** service.
6. Two services `UPDATE` the same row as policy?
   → wrong write owner, not a new folder. One writer; the other calls that service.

Owner: whose tests break if this rule is deleted? That package owns the file.

MUST NOT: `utils/`, `helpers/`, `common/`, `modules/common/`, `modules/shared/`, `modules/core/`.
MUST NOT: move a helper into `shared/` because a second caller appeared — add a service method, and a DTO if needed.

ADAPTED (GİRVAK, vidinsight-blog-service): one exception to #4, and only under
all three conditions — an **input-validation rule with no product noun**, needed
by **two or more modules' `schemas.py`**, where the service-call answer is
structurally impossible. A Pydantic field validator has no session and cannot
call another module's service, and *Kinds* forbids `schemas.py` importing
another module at all, so #4 has no reachable form. Such a rule stays in
`shared/` as a named policy file.

The one instance is `shared/url_policy.py` — an http(s) scheme allowlist for any
stored URL that will land in an `href`/`src`, used by `posts` and `authors`
schemas. It names no product noun, and copying it into both modules is what
#4 forbids anyway.

MUST NOT: read this as re-opening `shared/`. Everything with a product noun
still leaves: `slugify`, `tag_policy`, `markdown_policy`, `search_pattern` and
`author_social` all moved to their owners, and `tag_policy` moved **behind
`TagService`** because `posts` needed it at service time, where a service call
does work.

---

## Files (`orders`)

Day one — do not add a fourth file because the topic exists:

```
src/modules/orders/
  __init__.py    # empty. MUST NOT: re-export OrderService
  router.py      # prefix="/v1/orders". Paths, status, permission. OrderService(session)
  schemas.py     # OrderCreateRequest, OrderResponse
  service.py     # class OrderService. Helpers still in this file
```

Same noun, not this folder:

```
src/infra/db/models/order.py
src/infra/db/repositories/order.py
src/http/router.py                 # include — one line. MUST NOT: module registers in main.py
tests/modules/orders/              # mirror
```

Add a file only when its trigger is true. Do not scaffold the list.

- `errors.py` — 3+ feature exceptions. Until then: in `service.py`. MUST NOT: `errors/` folder (05).
- `dto.py` — another module, or a worker after session close, imports a non-HTTP shape. Until then: no file.
- `<work>.py` (`totals.py`, `render.py`, `policy.py`) — `service.py` failed 01. Not because "orders have totals". Not because a second URL exists.
- `deps.py` — constructor needs Redis, queue, or extra sessions. Until then: factory in `router.py`.
- Second router file — `router.py` itself failed 01. MUST NOT: `routes/`, `api/`.
  - ADAPTED (GİRVAK): also when the module serves more than one **audience**. An `APIRouter` carries one prefix, and the URL adaptation above gives one prefix per audience, so `modules/posts/` has `router.py` (`/v1/admin/posts`, editor JWT), `public_router.py` (`/v1/public/…`, no credentials, published rows only) and `ingest_router.py` (`/v1/ingest/posts`, service key). This is not the 01 size split and must not be used as a substitute for it: the test is that the files answer to **different permissions**. Merging them would put every reader-facing change in the file that grants admin access. `schemas.py` splits the same way and for the same reason — `posts/schemas.py` is the admin contract, `posts/public_schemas.py` the reader one — which is what lets the admin shape gain a field without it reaching the public API.

MUST NOT in this folder, ever:

- `models.py` `repository.py` — `infra/db/` (06)
- `services/` `*_service.py` a second `*Service`
- `helpers.py` `utils.py` `common.py` `helpers/`
- `v1.py` `api.py`

---

## Kinds

Stop at the first match. MUST NOT: `mapper`, `usecase`, `manager`.

### `router.py`

Is: HTTP for this resource.
Holds: `APIRouter(prefix="/v1/orders")`, permission, status, `OrderService(session)` from `http/deps` (session + current user).
Maps: request schema → service → response schema.
MUST NOT: SQL, `commit()`, `HTTPException`, repositories, helpers. Handler docstring: status + permission only. `Raises` on the service (01, 05).

### `schemas.py`

Is: HTTP contract for **this** router. Versioned with the URL, not the table.
Holds: `OrderCreateRequest`, `OrderResponse`.
Used by: this `router.py` only.
MUST NOT: another package import this file. MUST NOT: the service return a schema type.
MUST NOT: `schemas/` folder until 01 split.

### `service.py`

Is: the only public facade. Sentences: "cancel if unpaid".
Holds: one class `<Noun>Service`. Use cases (`create`, `cancel`, `get`). Repos from the session. Helpers. Other modules' **services**. `commit()`. Feature errors.
Returns: ORM while the session is open, or a DTO if `dto.py` exists.
Called by: this `router.py`, `workers/jobs/`, other modules' services.
MUST NOT: SQL strings, `HTTPException`, `OrderResponse`, a second `*Service`.

A query that spans two tables is **not** an exception to "no SQL here". It is a read repository in `infra/db/repositories/` returning a DTO; the service calls it like any other repo (06).

### `dto.py`

Is: a shape another **module** (or a worker after session close) must import. Not HTTP. Not ORM.
Open only when that import exists.
Holds: `OrderRef` / fields the collaborator needs. No OpenAPI aliases.
MUST NOT: a copy of `OrderResponse`. MUST NOT: router-only shapes (those are schemas).

### Helper `<work>.py`

Is: a slice of this noun the service calls. Not a facade.
Day one: no file — the work is still in `service.py`.
Name: the work (`totals.py`). Class if needed: `OrderTotals`, not `TotalsService`.
Holds: values in → values out. MAY: feature errors for this slice. The **service** reads `config/` and passes the number (`compute_totals(items, tax_rate)` — 03). MUST NOT: `get_settings()` / `os.getenv` in the helper — a helper test must not need an environment.
MUST NOT: session, repository, `commit()`, schemas, `get_current_user`, other modules' services, vendor I/O. Service loads; helper computes; service writes.

How given — no DI container, no `Depends`:

1. Function (default): `service.py` imports `compute_totals` and calls it with values it already has (`items`, `tax_rate` from `config/`).
2. Object (only if state: tax table, templates): `OrderService.__init__` does `self._totals = OrderTotals()` — no session, no `get_settings()`.

Router and workers construct `OrderService(session)` only. Other packages MUST NOT import the helper — they call `OrderService` (see Used by more than one module).

### Model / repository

Not in the module. `infra/db/models/`, `infra/db/repositories/` (06).

Two repository kinds (both live under `repositories/`, neither in the module):

- **Write** — one class ↔ one ORM model. `flush`; returns ORM / `None` / scalar. No `commit()`. No feature errors (service raises). No `join` of a second model.
- **Read** — only when one SQL must span two tables for a screen (06 trigger). Returns a frozen DTO, not ORM. `select` only — no `add` / `flush` / `commit`.

MUST NOT: SQL in `service.py` as a third kind.

---

## Imports

`router.py` → this `schemas`, this `service`; `http/deps`. MUST NOT: repos, helpers. Map DTO → schema only if the service already returned a DTO.

`service.py` → this helpers / errors / dto; other modules' `service` (+ `dto` if it exists); `infra/db/repositories` (write **and** read repositories, 06); `infra/cache|queue|storage` clients; `shared/errors` parents; `config/` values it was given.

`schemas.py` → Pydantic. MUST NOT: repos, helpers, other modules.

`<work>.py` → this errors; stdlib; `config/` if needed. MUST NOT: `infra/db`, other modules, `schemas.py`.

MUST: other packages import this `service` (and `dto` if present).
MUST NOT: other packages import this `router`, `schemas`, or a helper.

---

## Call chain

```
http/deps (session, user)
  → modules/<capability>/router.py
    → schemas (request)
    → <Noun>Service
         → helper(values)          # no I/O
         → repos / other services / infra clients
         → commit
    → schemas (response)
```

Workers skip router and schema: `workers/jobs/` → `<Noun>Service` ([11-workers.md](11-workers.md)). DTO before session close if the object must outlive it (06).

---

## Done

- [ ] Folder is the resource (`orders`), not `v1`; prefix is `/v1/orders`
- [ ] Day one is `__init__.py` + router / schemas / service unless a trigger fired
- [ ] One `*Service` in `service.py`; helpers named after the work; no session on helpers
- [ ] Model/write-repo/read-repo in `infra/db`; schema is HTTP; DTO only because a second package imports it (read-repo DTO stays next to that repo, 06)
- [ ] Two-module code has an owner (or infra/shared/http from Decide) — no `utils/` / `modules/common/`
- [ ] Other packages import service only (dto if needed)
