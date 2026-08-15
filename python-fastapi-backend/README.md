# Python FastAPI backend

WHEN: any Python edit in this stack (FastAPI HTTP process, workers, `src/` under the app `backend/` tree).
LOAD: [AGENTS.md](../AGENTS.md) first. Then [agents/02-turn.md](../agents/02-turn.md). Then this map. Then **one** numbered file below that matches the task (+ that file's `LOAD:` line only). On session start also LOAD [agents/01-boundary.md](../agents/01-boundary.md). On error LOAD [agents/04-errors.md](../agents/04-errors.md).

MUST NOT: load all sixteen files for one change.
MUST NOT: load Extra unless a line under Extra matches what this product **already** has.
MUST NOT: load anything under `for-humans/` unless `@`-referenced.

This playbook lives at `coding-playbook/python-fastapi-backend/`. The **application** tree it describes is still named `backend/` (02).

TARGETS: FastAPI with **Pydantic v2** and **SQLAlchemy 2.0** style (`Mapped[...]`, `select()`), Alembic, Python 3.11+. WHEN: the product is on an older major, or one of these ships a new major — the rules that encode framework behaviour (`05`, `06`, `07`, `10`, `12`, `13`) are the ones to re-read and adapt in git ([AGENTS.md](../AGENTS.md), *Not carved in stone*). MUST NOT: apply a rule from this playbook that the project's installed major does not have.

---

## Agent: which numbered file WHEN

WHEN: any Python edit and `01` not yet internalized this session.
LOAD: [01-coding-principles.md](01-coding-principles.md).

WHEN: new file and unsure which folder.
LOAD: [02-file-structure.md](02-file-structure.md).

WHEN: adding a package, `pyproject.toml`, lockfile, Python/base-image version, an audit finding, or a framework upgrade.
LOAD: [02-file-structure.md](02-file-structure.md) Dependencies + [agents/03-anti-patterns.md](../agents/03-anti-patterns.md) Dependencies **before** the manifest edit.

WHEN: env var, limit, timeout, secret name, feature flag.
LOAD: [03-config.md](03-config.md).

WHEN: log line, logger, filter, handler, startup logging.
LOAD: [04-logging.md](04-logging.md).

WHEN: exception class, raise site, error JSON mapping.
LOAD: [05-errors.md](05-errors.md).

WHEN: model, repository, session, commit, locking.
LOAD: [06-database.md](06-database.md).

WHEN: Alembic revision, add/change/drop column or table.
LOAD: [06-database.md](06-database.md) and [07-migrations.md](07-migrations.md) **same turn**.

WHEN: new infra package (cache, queue, storage) or "is this infra?".
LOAD: [08-infra.md](08-infra.md).

WHEN: module package, service, schema, helper, cross-module rule.
LOAD: [09-modules.md](09-modules.md).

WHEN: `http/router.py`, middleware, deps, mount list — not business rules.
LOAD: [10-http.md](10-http.md).

WHEN: job, worker, queue publish, retry, DLQ — not HTTP.
LOAD: [11-workers.md](11-workers.md).

WHEN: public URL, status code, list page, `error_code`, trace id.
LOAD: [12-api.md](12-api.md).

WHEN: JWT, login, authz, Redis identity, GraphQL/gRPC entry.
LOAD: [13-identity-security.md](13-identity-security.md) and [15-security.md](15-security.md).

WHEN: tests, mirror path, Postgres vs fakes.
LOAD: [14-testing.md](14-testing.md).

WHEN: PR review, upload, webhook, secret, auth change before merge.
LOAD: [15-security.md](15-security.md).

WHEN: pool, timeout, cap, cache, slow query, split decision.
LOAD: [16-performance.md](16-performance.md) and [03-config.md](03-config.md).

WHEN: user pasted build/runtime/API error.
LOAD: [agents/04-errors.md](../agents/04-errors.md) first; then the numbered file named in the matched block.

MUST NOT: open `RELATED:` unless the task is also that topic.

## Backbone

```
config/     typed settings. every env var and operational number.
http/       transport. no business rules.
modules/    one capability per package. routers live here.
infra/      DB models, repositories, cache, queue, storage.
workers/    call modules/*/service.py. no SQL.
```

MUST: `http/router.py` only mounts module routers.
MUST: ORM models and repositories stay in `infra/db`.
MUST NOT: process-local user state. Persist in DB, Redis, or object storage.

## How to load

Each numbered file carries two lines:

- `LOAD:` — what you must have open to do this task correctly.
- `RELATED:` — siblings to open **only if the task is also that topic**.

## Files

1. [01-coding-principles.md](01-coding-principles.md) — how to write
2. [02-file-structure.md](02-file-structure.md) — where the file lives; repo root files and tooling
3. [03-config.md](03-config.md) — settings object, env vars, secrets, where every limit lives
4. [04-logging.md](04-logging.md) — log line, logger, filter, handler, startup
5. [05-errors.md](05-errors.md) — exception class, raise site, HTTP error JSON
6. [06-database.md](06-database.md) — model, repository, session, commit, cross-model reads, locking
7. [07-migrations.md](07-migrations.md) — Alembic revision, expand/contract, locks
8. [08-infra.md](08-infra.md) — when code is infra, which folder, when to add a folder
9. [09-modules.md](09-modules.md) — module files, kinds, helpers
10. [10-http.md](10-http.md) — shell, mount list, deps vs auth, middleware
11. [11-workers.md](11-workers.md) — job vs request, runner, payload, retry/DLQ
12. [12-api.md](12-api.md) — success/error JSON, status, error_code, request_id trace
13. [13-identity-security.md](13-identity-security.md) — JWT, account, Redis, authz, GraphQL/gRPC
14. [14-testing.md](14-testing.md) — mirror path, what to test, Postgres vs fakes
15. [15-security.md](15-security.md) — layers that must not be skipped; PR pass
16. [16-performance.md](16-performance.md) — pools, timeouts, caps, when not to cache or split

Order is the order a process starts: how to write → where it goes → config → logging → errors → the layers → tests, security, performance.

## Extra

If the product **already** has that shape, LOAD the matching file (it names its own 01–16 siblings). 01–16 still apply. Extra **adds**; it does not replace `http/` · `modules/` · `infra/` · `workers/`.

MUST NOT: load Extra because it might be needed later. MUST NOT: scaffold `src/extra/`, `src/ws/`, `src/agents/` (or any Extra folder) because these files exist.

1. Two or more customers must not see each other's rows — [extra/01-multi-tenant.md](extra/01-multi-tenant.md)
2. More than one deployable backend (one repo or many) — [extra/02-microservices.md](extra/02-microservices.md)
3. The product runs an agent or an agent team — [extra/03-agent-teams.md](extra/03-agent-teams.md)
4. This backend imports a library you wrote (execution, agent kit, ML, …) — [extra/04-packages.md](extra/04-packages.md)
5. Clients get events without polling (WebSocket / SSE) — [extra/05-realtime.md](extra/05-realtime.md)
6. A DB write and a broker message must share one commit — [extra/06-outbox.md](extra/06-outbox.md)
7. Users sign in through an IdP (OIDC / SAML) — [extra/07-sso.md](extra/07-sso.md)
8. Ranked / engine search, not a filtered list — [extra/08-search.md](extra/08-search.md)
9. Customers register HTTPS webhook endpoints — [extra/09-webhooks.md](extra/09-webhooks.md)
10. Rows must be hidden, retained, or erased — [extra/10-retention.md](extra/10-retention.md)

Same list lives in [extra/README.md](extra/README.md) if you are already in that folder.
