# 14 · Tests

WHEN: adding or changing a test, a fixture, a factory, or deciding unit vs HTTP vs DB.
LOAD: this file only.
RELATED: 02 (tree) · 01 (header, one job) · 12 (HTTP contract) · 13 (auth fixtures) · 03 (test settings) — open only if the task is also that topic.
SCOPE: `backend/tests/`. Production `src/` is not a test dump.

The path **mirrors** `src/` (drop `src/`). Markers (`db`, `http`) are extra. MUST NOT: `tests/unit/` vs `tests/integration/` as the directory taxonomy (02).

---

## Where the file goes

```
src/modules/orders/service.py      →  tests/modules/orders/test_service.py
src/modules/orders/router.py       →  tests/modules/orders/test_router.py
src/modules/orders/totals.py       →  tests/modules/orders/test_totals.py
src/infra/db/repositories/order.py →  tests/infra/db/repositories/test_order.py
src/http/errors/                   →  tests/http/errors/test_handlers.py
src/workers/jobs/issue_invoice.py  →  tests/workers/jobs/test_issue_invoice.py
```

Name: `test_<src_file>.py` next to the same folders. MUST NOT: `test_orders.py` that mixes router + service + repo.

`tests/conftest.py` — engine, session scope, ASGI client, clock. File-local fixtures stay in that test file.

`tests/factories/` — open only when **three** test packages build the same graph. Named after the noun (`order.py`, `user.py`). MUST NOT: `helpers.py`. Until then: a builder function in the module's test package.

Tests for `shared/` / `config/` only when that package has behavior. Do not create empty mirror folders.

---

## What to test (minimum)

For each **public** service method: one happy path, one edge (empty list, already-cancelled), one typed error (`pytest.raises(OrderNotFoundError)`).

For each **public** HTTP route: status + body shape (12). At least: 2xx happy, 401 without Bearer (if protected), 404 missing-or-not-owned (same `error_code`), 422 validation. MUST NOT: assert on `message` text as the contract — assert `error_code` and status.

For a **repository**: SQL the service actually calls (`get_by_id`, `create`). Real test Postgres. MUST NOT: mock `AsyncSession` to "unit test" SQL.

For a **helper** (`totals.py`): the rule, no session.

For a **job file**: parse payload → calls the service with those ids. Mock the queue. Rule tests stay on the service (11).

MUST NOT: a test whose only caller-setup is to cover a line of FastAPI/SQLAlchemy. MUST NOT: a repository method that exists only so a test can call it (06).

---

## Name and shape

`async def test_<action>_<condition>_<result>(…) -> None:`

MUST NOT: `test_create`, `test_orders`. MUST: the name is the assertion.

One test = one behavior. MUST NOT: `test_create_and_cancel`.

Arrange → act → assert. MUST NOT: `# Arrange` comments that restate the next line (01). A `# freeze time — expiry is 03:00Z` is allowed.

File header (01): `Layer: Test`. `Called by: pytest`. `Calls:` the module under test.

pytest-asyncio: `asyncio_mode = auto`. MUST NOT: decorate every test with `@pytest.mark.asyncio` if auto is on. Mark `@pytest.mark.db` when the test needs Postgres.

---

## DB, HTTP, fakes

Test database is **PostgreSQL** (same JSONB/async as 06). MUST NOT: SQLite "for speed" — it lies. MUST NOT: production DSN.

Provision it with testcontainers or a compose service, once per test session. Build the schema with `alembic upgrade head` (07), not `metadata.create_all` — otherwise the revisions are never exercised and the test schema drifts from production.

### Isolation

The service commits (06). A fixture that only calls `session.rollback()` at teardown therefore isolates **nothing** — the rows are already committed and leak into the next test. Pick one of the two patterns below and stay.

**1. Savepoint-bound session (default).** The test owns an outer transaction; the session joins it as a savepoint, so the service's `commit()` releases the savepoint while the outer transaction is still rolled back at teardown.

```python
@pytest_asyncio.fixture
async def session(engine: AsyncEngine) -> AsyncIterator[AsyncSession]:
    async with engine.connect() as conn:
        await conn.begin()
        factory = async_sessionmaker(
            bind=conn,
            expire_on_commit=False,
            autoflush=False,
            join_transaction_mode="create_savepoint",
        )
        async with factory() as s:
            yield s
        await conn.rollback()          # undoes everything the service committed
```

MUST: the code under test uses **that** session. For HTTP tests that means overriding the session dependency:

```python
app.dependency_overrides[get_session] = lambda: session
```

MUST NOT: use this pattern when the code under test opens its own connection (a worker runner, a second engine, anything with `SELECT … FOR UPDATE` across sessions). It will not see the uncommitted outer transaction.

**2. Truncate between tests.** Commits are real; a fixture wipes the tables afterwards.

```python
await conn.execute(text(
    "TRUNCATE TABLE orders, users RESTART IDENTITY CASCADE"
))
```

MUST: use this when the test spans two sessions or two processes, and when the assertion is about commit/rollback behaviour itself (concurrency tests, 06).
MUST: truncate every table, generated from `Base.metadata.sorted_tables` — a hand-written list rots the first time a table is added.

MUST NOT: mix the two in one suite without saying which fixture a test uses.
MUST NOT: test order dependence. Each test passes when run alone (`pytest <file>::<test>`).

HTTP: `httpx.AsyncClient` against the ASGI app. Hits real routers + services + test DB. Auth: fixture issues a JWT with the test secret (13), or `dependency_overrides` only when the test is about a **downstream** layer, not about auth itself.

Fakes:

- Redis / queue / email / payments — in-memory fake or testcontainer. MUST NOT: prod Redis, SendGrid, Stripe.
- Service tests **may** use real repos + test DB (preferred: sees flush/commit). Mock a repo only when the test is the service rule and the SQL is already covered under `tests/infra/`.
- MUST NOT: mock `OrderService` in `test_service.py`. MUST NOT: mock the handler in `test_handlers.py`.

Time: inject a clock or freeze; MUST NOT: `sleep(2)` for expiry.

---

## Isolation and data

Each test builds the rows it needs (factory or explicit `User` + `Order`). MUST NOT: a session-scoped "seed the world" that later tests mutate.

Factories return persisted instances on the **same** session the service will use, or ids after commit — pick one per fixture and stay. MUST NOT: pass an ORM from session A into session B (06).

Secrets in fixtures are obviously fake (`test-jwt-secret`, not prod). MUST NOT: copy `.env` production values into `conftest.py`.

---

## CI

`pytest` from `backend/`. Default suite is the mirror. A slower mark (`db`) runs in CI always; local may filter.

SHOULD: coverage on `src/modules/` (and the slice you changed). MUST NOT: fail the playbook on covering `main.py` glue or generated Alembic.

A failing test is not fixed by deleting the assertion or marking `xfail` without a ticket id in the mark reason.

---

## Done

- [ ] Path mirrors `src/…`; not `tests/unit/` / `tests/integration/` as folders
- [ ] Name states action, condition, result; one behavior
- [ ] Public service method has happy / edge / typed error
- [ ] Public route asserts status + `error_code` (12), not `message` copy
- [ ] Postgres test DB, schema from `alembic upgrade head`; no SQLite, no prod DSN, no prod Redis
- [ ] Isolation is savepoint-bound **or** truncate — not a bare `rollback()` against a service that commits
- [ ] Test passes when run alone
- [ ] Did not mock the unit under test
