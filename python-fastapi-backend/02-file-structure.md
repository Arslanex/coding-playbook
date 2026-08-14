# 02 · File structure

WHEN: creating a file, folder, or deciding where code belongs.
LOAD: this file only. Then the numbered file that Decide points at.
RELATED: 01 (how to write) · 03 (what goes in `config/`) — open only if the task is also that topic.
SCOPE: the application `backend/` tree this stack describes. Do not invent a parallel tree. This playbook folder is `python-fastapi-backend/`; do not name the app after it.

MUST: first matching yes in Decide. MUST NOT: add a top-level folder not in the tree.
MUST NOT: `utils/` · `helpers/` · `common/` anywhere. A helper lives in the module that uses it.
MUST NOT: copy [extra/](extra/README.md) into `src/` — Extra is playbook, loaded only when that need is real.

---

## Tree

Directories. Leaves shown in the tree are fixed (`main.py`, `session.py`, `http/router.py`, `http/deps.py`). Other names: the numbered file for that folder.

```
backend/
├── pyproject.toml
├── alembic.ini
├── .env.example
├── src/
│   ├── main.py
│   ├── config/
│   ├── shared/
│   │   ├── errors/
│   │   └── logging/
│   ├── http/
│   │   ├── router.py
│   │   ├── deps.py
│   │   ├── middleware/
│   │   └── errors/
│   ├── infra/
│   │   ├── db/
│   │   │   ├── session.py
│   │   │   ├── models/
│   │   │   ├── repositories/
│   │   │   └── migrations/
│   │   ├── cache/
│   │   ├── queue/
│   │   └── storage/
│   ├── modules/
│   │   └── <capability>/
│   └── workers/
│       └── jobs/
└── tests/
    ├── modules/
    ├── http/
    ├── infra/
    └── workers/

# MUST NOT exist: utils/  helpers/  common/
```

Helpers exist. A `utils/` folder does not. Named file in the owning module (`totals.py`), or `infra/` / `shared/` by Decide. Two callers: [09-modules.md](09-modules.md).

---

## Decide

Unknown unit: ask in order. Stop at the first yes.
Product noun = `orders`, `auth`, `billing`. Not `service`, `api`, `util`.

1. Exists only because an external system exists (Postgres, Redis, queue, S3, Stripe/SendGrid HTTP)?
   → `infra/` then the matching child. Details: [08-infra.md](08-infra.md).
2. Business rule, or a product HTTP route/schema about a product noun?
   → `modules/<capability>/`. Details: [09-modules.md](09-modules.md).
3. Runs on every request as transport (request id, metrics, rate limit, session, current user, error JSON)?
   → `http/` then the matching child. Details: [10-http.md](10-http.md).
4. Background process that consumes a job payload?
   → `workers/jobs/`. Details: [11-workers.md](11-workers.md).
5. Kernel primitive with no product noun (base exception class, logger wiring)?
   → `shared/`
6. Environment → typed settings?
   → `config/`. Details: [03-config.md](03-config.md).
7. Test?
   → `tests/` mirroring `src/` (drop the `src/` segment). Details: [14-testing.md](14-testing.md).

No yes: not a new top-level folder. Put a helper file inside the existing `modules/<capability>/` (01: when to split).
MUST NOT: open `shared/` because two modules need it. Owner + call the service: [09-modules.md](09-modules.md) (Used by more than one module). If only one module uses it, it stays there.

---

## Root files

Four files at `backend/`, no more. A new dotfile at the root needs a reason a reviewer would accept.

`pyproject.toml` — the only place for dependencies **and** tool config. One `[project]`, one pinned Python version (`requires-python`), a lockfile committed next to it. MUST NOT: `requirements.txt` alongside it as a second source of truth. MUST NOT: an unpinned direct dependency.

Tool sections live here too, so there is no `setup.cfg` / `.flake8` / `mypy.ini` / `pytest.ini` scattered around:

- `[tool.ruff]` — lint **and** format. One tool. Import order (01) is `ruff` rule `I`, enforced, not a convention agents remember.
- `[tool.mypy]` — typed values across boundaries (01) only mean something if they are checked. Strict on `src/`; tests may be looser.
- `[tool.pytest.ini_options]` — `asyncio_mode = "auto"`, markers, test paths (14).

MUST: lint, format-check, type-check and tests all run in CI on the same command set a developer runs locally. A rule that only exists in prose is not enforced.
MUST NOT: silence a type error with a bare `# type: ignore` — narrow it (`# type: ignore[arg-type]`) or fix the type.

`alembic.ini` — at the **backend root**, not inside the migrations folder. `script_location` points at `src/infra/db/migrations` (07). MUST NOT: a second Alembic config for a "test" chain.

`.env.example` — names of every required setting, empty values, committed. `.env` itself is git-ignored (03, 15).

---

## What each folder is

### `src/main.py` — HTTP process entry

Is: the API app factory. Wires middleware order and lifespan (open/close pools). Worker process entry is `workers/runner.py` (11).

Put here: adding a middleware to the global chain, or startup/shutdown of infra clients for the **API** process.

Put elsewhere: a rule or a route → `modules/`. SQL → `infra/db/`. Consume loop → `workers/runner.py`.

### `src/config/` — settings

Is: env vars turned into one typed settings object. How it is written: [03-config.md](03-config.md).

Put here: nested settings fields (03) — `DATABASE__DSN`, `DATABASE__POOL_SIZE`, `REDIS__URL`, secret name, operational numeric limit. Env names follow the object path, not a flat `DATABASE_URL`.

Put elsewhere: business policy ("cancel only if unpaid") → `modules/`.
MUST NOT: `os.getenv` / `os.environ` outside `config/` — not in `modules/`, `infra/`, `http/`, `workers/`, or `main.py` (03).

### `src/shared/` — kernel, no product noun

Is: code that would still exist if the product had zero features.

`shared/errors/` — base exception types every feature error subclasses.
`shared/logging/` — logger setup and log context. Not "log that an order cancelled".

Put here: a base class or logger adapter with no `orders` / `auth` in the name.

Put elsewhere: `OrderNotCancellableError` → that module. HTTP status mapping → `http/errors/`.
MUST NOT: grow `shared/` into a toolbox.

### `src/http/` — transport shell

Is: this process speaks HTTP. It does not know orders. Details: [10-http.md](10-http.md).

Put here: behavior true for all routes (or all authenticated routes) and names no product noun.

Put elsewhere:
- login, password, tokens as a product → `modules/auth/`
- `OrderService(...)` factory → that module, not `http/deps`
- `raise OrderNotFoundError` → module; only the JSON mapping is `http/errors/`
- a new URL for orders → `modules/orders/`, then one mount line in `http/`

MUST NOT: `http/v1/` folder. Version is a prefix on the module router (`/v1/...`).
MUST NOT: `src/api/` as a second tree. Day-one identity file is `deps.py`, not `deps/auth.py`.

### `src/infra/` — adapters to the outside

Is: how we talk to a system we do not own the meaning of.

When code belongs here, which child folder, when to open a new folder, naming: [08-infra.md](08-infra.md).

`infra/db/` — PostgreSQL. Details: 06, 07.
`infra/cache/` `infra/queue/` `infra/storage/` — primitives. Details: 08.

MUST NOT: business rules, HTTP routes, or `commit()` in repositories.


### `src/modules/<capability>/` — one product ability

Is: a product noun and everything that means.

Template, when to add a file, naming: [09-modules.md](09-modules.md).

Put here: deleting the folder would remove a user-visible ability.
Put elsewhere: SQL → `infra/db/` (06). Vendor HTTP → `infra/` (08). Current user → `http/deps`. Job loop → `workers/jobs/`.

### `src/workers/` — out-of-request process

Is: a consumer. Reads a payload, calls a module service, exits. Stateless. Details: [11-workers.md](11-workers.md).

Put here: the work must not run inside the HTTP request (slow, retryable, fan-out). Consume via `infra/queue/` (08). MUST NOT: worker → repository.

### `tests/` — mirror

When to write which test, naming, DB vs fake: [14-testing.md](14-testing.md).

`tests/modules/` ↔ `src/modules/`
`tests/http/` ↔ `src/http/`
`tests/infra/` ↔ `src/infra/`
`tests/workers/` ↔ `src/workers/`

Put here: the test asserts that folder.
MUST NOT: replace this mirror with a different taxonomy (`unit/` vs `e2e/` as the path). Extra markers are allowed; the path still mirrors.

---

## Call chain

```
modules/<capability>/  (route)  →  modules/<capability>/  (service)  →  infra/db/repositories/
workers/jobs/                   →  modules/<capability>/  (service)
```

MUST NOT: route → repository
MUST NOT: worker → repository
MUST NOT: repository `commit()` — service owns the transaction
MUST NOT: `http/` import a module service except the mount list importing the router object

---

## Done

- [ ] Decide first-yes matches the path
- [ ] Folder "Is" still true; "Put elsewhere" was not ignored
- [ ] No `utils/` / `helpers/` / `common/`
- [ ] Test path mirrors `src/`
- [ ] Call chain still holds
- [ ] Root has `pyproject.toml` + `alembic.ini` + `.env.example` and no rival config file
