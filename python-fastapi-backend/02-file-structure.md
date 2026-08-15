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
│   │   │   └── repositories/
│   │   ├── cache/
│   │   ├── queue/
│   │   └── storage/
│   ├── modules/
│   │   └── <capability>/
│   └── workers/
│       └── jobs/
├── migrations/              # Alembic chain. beside src/, not in it (07)
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
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

0. An Alembic revision, `env.py`, or `script.py.mako`?
   → `migrations/` **beside** `src/`, not inside it. Details: [07-migrations.md](07-migrations.md). It answers yes to 1 as well — this rule wins, because nothing in `src/` imports it.
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

`src/` stays uniformly strict because generated code is not in it. `migrations/versions/` is written by Alembic, and lint/type rules that are right for hand-written code are noise there — keep it out of `[tool.mypy]`'s strict set the way `tests/` already is.

MUST NOT: exclude the whole `migrations/` folder. `env.py` and `script.py.mako` are hand-written and stay checked; `env.py` importing the wrong metadata is exactly the bug these tools catch (07).
MUST NOT: loosen strict mode for `src/` because a generated file complains. Nothing generated lives there.

MUST: lint, format-check, type-check and tests all run in CI on the same command set a developer runs locally. A rule that only exists in prose is not enforced.
MUST NOT: silence a type error with a bare `# type: ignore` — narrow it (`# type: ignore[arg-type]`) or fix the type.

`alembic.ini` — at the **backend root**, beside the `migrations/` it points at. `script_location = migrations` (07). MUST NOT: a second Alembic config for a "test" chain.

`.env.example` — names of every required setting, empty values, committed. `.env` itself is git-ignored (03, 15).

---

## Dependencies

Pinning above is reproducibility. It is not the whole job — which package, which version, and staying current are also this file's, and none of them is "later."

**Adding one.** MUST: the package exists on the index, is the one you meant, and is maintained — check before writing the name into `pyproject.toml`, not after the import fails. MUST NOT: write a package name from memory. An invented or misremembered name is a name someone may have registered for exactly that mistake ([agents/03-anti-patterns.md](../agents/03-anti-patterns.md)).

MUST: prefer the standard library and what FastAPI/Pydantic/SQLAlchemy already give you. A slug, a retry loop, a UUID, a datetime format is not a dependency. Every added package is a supply-chain surface (15) and a thing to keep current forever.

MUST: CI installs **from** the committed lockfile (the frozen/sync install, not a resolving one). MUST NOT: regenerate the lockfile to make an error go away.

**Choosing the version.** MUST NOT: write a version number from memory. The version you remember is the one current while you were trained — by definition old, and old is where the published advisories are. A correct package name with a stale version passes every rule above.

HOW: let the tool resolve it — `uv add <pkg>` / `poetry add <pkg>` writes the current version and updates the lockfile together. If `pyproject.toml` must be edited by hand, read the current version from the index first.
MUST NOT: copy a version from a tutorial, an answer, or another project's `pyproject.toml`.
MUST NOT: guess a range around a remembered number.
MUST: applies hardest to `fastapi`, `pydantic`, `sqlalchemy`, `alembic` — the ones you are most confident about.

**Keeping current.** An outdated dependency with a known advisory is a security finding, not backlog.

MUST: a dependency audit step runs in CI on every PR. MUST: a high-severity advisory on a package that actually ships blocks merge — fix, upgrade, or write the accepted risk and its expiry in the repo.
MUST: the base image and the pinned Python are upgraded on a schedule with an owner (08). A pinned old runtime is a CVE list that stops moving, not stability.
MUST NOT: pin a version **forever** to avoid a migration. Pinning is for reproducible builds, not for skipping upgrades.

Tooling is the project's choice (an audit command, a bot that opens upgrade PRs, a scheduled job) — the rule is that the step exists and a finding has an owner.

---

## Build and release

WHEN: the user asks for a container image, a CI pipeline, or a release step.
MUST NOT: write any of these unasked ([agents/01-boundary.md](../agents/01-boundary.md)). This section says what they contain **when** asked, not that they should exist.

`Dockerfile` at the backend root. Multi-stage: one stage installs from the lockfile, the final stage copies `src/` **and** `migrations/` (07) and nothing else.

MUST: a base image pinned by digest, not a floating tag. `python:3.12-slim` moves under you; the digest does not, and the upgrade becomes a visible diff (02 *Dependencies*).
MUST: run as a non-root user.
MUST: `.dockerignore` excluding `.env`, `.git`, `tests/`, caches. MUST NOT [critical]: a secret baked into a layer — layers are extractable from any pulled image, and deleting the file in a later layer does not remove it.
MUST: the same image serves the API and the workers — different **command**, not a second build (11).
MUST: a health endpoint the platform can call that does **not** hit the database on every probe.

MUST NOT: `alembic upgrade head` in the image entrypoint. Migrations are a release job, and two pods starting at once would race (07 *Runtime*).

CI, in this order: install from the lockfile → lint → type-check → tests against a real Postgres (14) → dependency audit (02). Same commands a developer runs locally.
MUST NOT: a pipeline that runs migrations against production without a human step.

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

`infra/db/` — PostgreSQL session, models, repositories. Details: 06. The revision chain is **not** here — it is `migrations/` beside `src/` (07).
`infra/cache/` `infra/queue/` `infra/storage/` — primitives. Details: 08.

MUST NOT: business rules, HTTP routes, or `commit()` in repositories.


### `src/modules/<capability>/` — one product ability

Is: a product noun and everything that means.

Template, when to add a file, naming: [09-modules.md](09-modules.md).

Put here: deleting the folder would remove a user-visible ability.
Put elsewhere: SQL → `infra/db/` (06). Vendor HTTP → `infra/` (08). Current user → `http/deps`. Job loop → `workers/jobs/`.

### `src/workers/` — out-of-request process

Is: a consumer. Reads a payload, calls a module service, exits. Stateless. Details: [11-workers.md](11-workers.md).

Put here: the work must not run inside the HTTP request (slow, retryable, fan-out). Consume via `infra/queue/` (08). What a worker may call: *Call chain* below.

### `migrations/` — schema history, beside `src/`

Is: the Alembic chain. `env.py`, `script.py.mako`, `versions/`. How to write a revision: [07-migrations.md](07-migrations.md).

Sits beside `src/` for the same reason `tests/` does: the application never imports it, and it runs from its own command in a release job, not from the app process (07, Runtime). `src/` is what gets imported and shipped as the package; a revision is history that a migrate step replays against a database.

Put here: anything Alembic reads or writes.
MUST NOT: `src/infra/db/migrations/`. `infra/db/` owns the session, models, and repositories the app imports — the revision chain is not one of them.
MUST NOT: a second chain (per-module, or a "test" one). One chain per database (07). Several services each owning tables: [Extra 02](extra/02-microservices.md).

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
- [ ] New package: verified on the index, version resolved by the tool — no hand-typed version
- [ ] Alembic chain is `migrations/` beside `src/`, not under `infra/db/`
- [ ] `src/` has no exclusions; `versions/` is out of the strict set, `env.py` is not
