# 06 · Database

WHEN: adding a table, column, repository method, session, or commit.
LOAD: this file only.
RELATED: 02 (`infra/db/` placement) · 07 (the revision this change needs) · 13 (user/session tables) · 16 (pool sizing) — open only if the task is also that topic.
SCOPE: PostgreSQL + SQLAlchemy 2.0 async (`AsyncSession`, `Mapped`).

The database is an adapter. Meaning lives in `modules/`. SQL lives in `infra/db/`.

---

## Folder

```
infra/db/
  session.py         # engine + pool + sessionmaker + scope (open / rollback / close / dispose)
  models/            # DeclarativeBase, mixins, tables
    base.py
  repositories/      # thin Base + one class per model
    base.py
  migrations/        # Alembic
```

MUST NOT: `uow.py`. MUST NOT: `engine.py` as a sibling — the engine stays inside `session.py` until 01 split rules fire.
MUST NOT: a `UnitOfWork` class. The `AsyncSession` is the unit of work.

MUST: `AsyncSession` only. MUST NOT: sync `Session` inside `async def`.
MUST: credentials from `config/` / secret store. MUST NOT: DSN in logs, exceptions, or `details` (05).
MUST NOT: business rules, HTTP routes, or `commit()` in a repository.

`http/deps` (`deps.py`, 10) yields a session from `session.py`.
`workers/jobs/` opens the same scope around the service call (11).

---

## Session management

`session.py` is the only runtime file under `infra/db/`.

Engine (private to this file): one `AsyncEngine` for the process. Pool size from `config/` ([03-config.md](03-config.md)) — size it against in-flight sessions ([16-performance.md](16-performance.md)). `pool_pre_ping=True`. `dispose()` on API shutdown (`main.py` lifespan) and worker shutdown (`workers/runner.py`). MUST NOT: `create_async_engine` anywhere else.

`statement_timeout` is set **on the connection**, not per query — asyncpg takes it as a server setting at connect time:

```python
create_async_engine(
    settings.database.dsn,
    pool_size=settings.database.pool_size,
    max_overflow=settings.database.max_overflow,
    pool_pre_ping=True,
    connect_args={
        "server_settings": {
            "statement_timeout": str(settings.database.statement_timeout_ms),
        }
    },
)
```

MUST: a runtime `statement_timeout` exists. A query with no ceiling holds a pool slot forever (16). Migration timeouts are separate and live in the revision (07).

Session factory: `async_sessionmaker(engine, expire_on_commit=False, autoflush=False)`. `autoflush=False` so a read cannot silently flush a half-built row; the repo `flush()` is explicit.

Session scope (used by `http/deps` and workers):

```
enter:  open session
yield:  same session to the service (and its repos)
exit:   if the use case raised → rollback
        always close (return connection to the pool)
```

MUST NOT: commit in `session.py` or in `http/deps`. The service commits.

The `AsyncSession` **is** the unit of work. MUST NOT: a `UnitOfWork` class.

HTTP:

1. `http/deps` calls the session scope, yields `AsyncSession`.
2. Router builds `OrderService(session)`.
3. Service method runs (repos share that session).
4. Service `await session.commit()` on success.
5. Scope closes. If step 4 never ran because of an exception, rollback then close.

Worker: the same scope around `await service.…()`. MUST NOT: a long-lived session for the whole worker process.

MUST: one session per request or job. MUST NOT: nest a second session for the same use case.
MUST NOT: store the session on a module global or singleton. MUST NOT: a repository or service open its own session — inject it.

After `commit()`, attributes stay loaded (`expire_on_commit=False`). MUST NOT: rely on that across a session close.

MUST: repository `add` + `flush` so the PK exists before commit. MUST NOT: repository `commit()` or `rollback()`.

Two use cases = two commits = two service methods, or one method that commits once at the end. MUST NOT: commit in the middle of a use case so a later failure leaves a half-write.

---

## Leak prevention

These are the leaks this layer must not produce.

**Connection leak.** Session is a context manager. Every path closes it (`finally` / `async with`). MUST NOT: `session.close()` forgotten on an error path.

**Greenlet / lazy-load leak.** Async cannot implicit-IO after `await`. MUST: `expire_on_commit=False` on the session factory. MUST: relationships `lazy="raise"` (or load them in the same query with `selectinload`). MUST NOT: `order.items` after an `await` unless loaded.

**Detached instance leak.** An ORM object is invalid after the session closes. MUST NOT: return ORM from a worker after the `async with session` block. Convert to a DTO before close if the object must outlive the session (fan-out, cache, queue payload).

**N+1 leak.** MUST NOT: loop `get_by_id` in a service. Add a `list_by_ids` on that repo when a caller needs it (not before).

**Identity leak.** MUST NOT: one session for the whole process. MUST NOT: pass an ORM from session A into session B.

**Credential leak.** Connection errors become `ServiceUnavailableError` (05). MUST NOT: include the DSN or SQL in `message` / `details`.

---

## Models

SQLAlchemy 2.0 declarative. One `DeclarativeBase` with a naming convention (`ix_`, `uq_`, `ck_`, `fk_`, `pk_`) so Alembic names stay stable.

### Mixins

Mixins are opt-in column bundles on the model class. They live in `models/base.py` next to `DeclarativeBase`. MUST NOT: `models/mixins/`.

Allowed from day one (every ordinary table uses both):

- `UUIDPrimaryKey` — `id UUID`, `gen_random_uuid()`
- `Timestamps` — `created_at` + `updated_at`, timezone-aware. `updated_at` uses Python `onupdate`, not a server `NOW()` that expires the column in async

```python
class Order(Base, UUIDPrimaryKey, Timestamps):
    __tablename__ = "orders"
    ...
```

MUST NOT: put `id` / timestamps on `DeclarativeBase` itself. Association tables and outbox rows often have a different PK.

MUST NOT: `AuditMixin`. `created_by` / `updated_by` is not kernel. Add those columns on the tables that have an actor. If five or more tables share the exact same pair, then one mixin named after the columns (`ActorColumns`), still in `base.py` — not "audit" (that name is the logger, 04).

MUST NOT: `SoftDeleteMixin`, `TenantMixin`, or any mixin with no caller yet (same rule as repository methods).

A mixin is only columns. MUST NOT: mixin methods that encode business rules.

MUST:

- nullable JSONB via `JSONB(none_as_null=True)` so Python `None` is SQL NULL
- columns typed with `Mapped[...]`
- FKs declared on the model; constraint names from the convention

MUST NOT:

- business methods on the model (`order.cancel()`). That is the service
- `lazy="select"` / default lazy on relationships in async
- mutable default `{}` / `[]` in Python — use `server_default` for JSONB
- `mapped_column(JSON)` when the database is PostgreSQL — use `JSONB`

A model file is table + relationships. Invariants stay in the module service.


---

## Base repository

`infra/db/repositories/` has a **thin** base. It is not a generic CRUD framework.

Base holds:

- the `AsyncSession`
- the model type the subclass is bound to
- `add(instance)` + `flush()` so subclasses do not reimplement that

Base does **not** hold: `list_all`, `paginate`, `update`, `delete`, `search`, `exists`, `count` — until a concrete repo needs one **and** the implementation is identical. Copying a four-line `get_by_id` is cheaper than a base method nobody calls.

MUST NOT: a Base that takes a `dict` and sets attributes generically. The service builds the model (or a typed method on the concrete repo does).

---

## Concrete repositories

One repository class ↔ one ORM model. `OrderRepository` may touch only `Order`.

MUST NOT: `select(Order, Customer)` or `join(Customer)` inside `OrderRepository`. The service calls `OrderRepository` and `CustomerRepository`. When one SQL statement genuinely must span both, that is a **read repository** (next section) — not a second model smuggled into the order repo, and never a SQL string in the service.

MUST NOT: two models in one repository class. Two tables = two repository classes. They may live in one file only while both stay small (01); they remain two types.

Constructor: `def __init__(self, session: AsyncSession) -> None`.

Returns: ORM instance, `list[Model]`, `None`, or a scalar. MUST NOT: `dict`. MUST NOT: raise `OrderNotFoundError` — return `None`; the service raises (05).

`create`: `session.add` + `flush`, return the instance. No commit.

Integrity errors (`IntegrityError`) propagate. The service translates to `ConflictError` / a feature type. The repository does not catch and remap.

---

## Cross-model reads (joins)

A list screen that needs columns from two tables has exactly one legal home. It is not the service (no SQL there — 09), not a helper (no session — 09), and not a write repository (one model).

`infra/db/repositories/` may hold a **read repository**: read-only SQL across models, returning a frozen DTO.

Trigger — all three must be true. Until then, two repository calls in the service is the answer.

1. The service would otherwise loop, or issue N queries for one screen
2. The rows come from two or more tables in one logical answer
3. Nothing is written on this path

```python
"""
Module: infra/db/repositories/order_summary.py
Layer: Repository
Purpose: Read-only order rows joined with customer name for the list screen.
         Writes nothing. Owns no rule.
"""

class OrderSummaryReadRepository:
    def __init__(self, session: AsyncSession) -> None: ...

    async def list_for_user(
        self, user_id: uuid.UUID, limit: int, cursor: str | None
    ) -> list[OrderSummaryRow]: ...
```

MUST: name it `<question>ReadRepository` and the file after the question (`order_summary.py`). MUST NOT: `QueryService`, `ReadService`, `views.py`.
MUST: return a frozen dataclass / Pydantic model (`OrderSummaryRow`) declared next to it. MUST NOT: return ORM instances from a join, and MUST NOT: return `dict`.
MUST: read only — `select` and nothing else. No `add`, no `flush`, no `commit`.
MUST: the caller is the owning module's service, which still applies authz and still owns the `WHERE user_id` / tenant filter it passes in (15, [Extra 01](extra/01-multi-tenant.md)).
MUST NOT: a read repository that spans capabilities nobody owns (`everything_dashboard.py`). If two modules want it, one module owns the screen and the other calls that **service** (09).
MUST NOT: raw SQL strings unless the query is impossible in the ORM expression language. If it is, keep it in this file with `text()` and bound parameters — never f-strings (15).

Write ownership is unchanged: `OrderRepository` still owns writes to `orders`. A read repository never becomes the place a second writer appears.

The router still calls the service; the service calls the read repository and maps its rows to the response schema (09, 12).

---

## Concurrent writes

Two requests can execute the same use case at the same time. "Check then write" in Python is not atomic — the check happened in another transaction's snapshot.

Stop at the first yes.

1. A uniqueness rule (one active subscription, one slug)?
   → **database constraint** (unique / partial unique index, 07). Let `IntegrityError` propagate; the service translates it to `ConflictError` (05). MUST NOT: `if await repo.exists(...)` as the only guard.
2. A state transition that must not run twice (cancel, capture, mark-paid)?
   → **conditional UPDATE**: `UPDATE … SET status='cancelled' WHERE id=… AND status='confirmed'`, then check `rowcount`. Zero rows = someone else won → raise the feature `ConflictError`. One statement, no race.
3. A counter or balance read-modify-written in Python?
   → do the arithmetic **in SQL** (`SET balance = balance - :amount WHERE balance >= :amount`), check `rowcount`. MUST NOT: `obj.balance -= x` after a plain `SELECT`.
4. A multi-row invariant that cannot be one statement (rebalance, allocate from a pool)?
   → `SELECT … FOR UPDATE` on the owning row, in the same transaction, ordered by `id` so two transactions cannot deadlock by locking in different orders. Keep the lock short — no vendor I/O while it is held (16).
5. Work triggered by a queue that may deliver twice?
   → idempotency on the **service** (11), backed by 1 or 2 above. MUST NOT: a Redis lock as the only guard — a lost lock is not a rolled-back write (13).

MUST NOT: `SELECT FOR UPDATE` as the default for every write. Most use cases are 1 or 2.
MUST NOT: raise the isolation level to `SERIALIZABLE` instead of writing the constraint — then every caller must handle serialization failures and nobody does.
MUST: a test that runs the two concurrent paths for money and for state transitions (14).

---

## Column types that bite

- **Money / any exact decimal** — `Numeric(precision, scale)` in Postgres, `Decimal` in Python. MUST NOT: `float` / `Float` for money. Store the currency next to it; MUST NOT: assume one currency in the column type.
- **Time** — `DateTime(timezone=True)`, always UTC in the database. Naive `datetime` never crosses a boundary. MUST NOT: `datetime.utcnow()` (naive); use an aware now, injected as a clock where a test must freeze it (14).
- **Status / enum** — `String` + a `CHECK` constraint, mapped to a Python `Enum`. MUST NOT: a native PostgreSQL `ENUM` type. Adding a value later is a locking `ALTER TYPE` on a hot table (07); a `CHECK` is a cheap constraint swap.
- **Id** — UUID PK (15). Random v4 scatters index writes; if a table is high-insert, prefer a time-ordered UUID (v7) so the PK index stays local. Either way the value stays opaque to the client. MUST NOT: a sequential integer as the public id.
- **Text** — `Text` unless a length is a real rule. A `String(255)` that means nothing is a future migration.
- **JSONB** — see Models above. MUST NOT: JSONB as a way to avoid designing a column that is queried.

---

## Add methods when called

A new repository starts with what the first caller needs. Usually `get_by_id` and `create`. Nothing else.

MUST: add `list_by_ids`, `list_for_user`, `delete` only when a service line would otherwise do N queries or raw SQL.

MUST NOT: scaffold `update`, `delete`, `list`, `count` "because it is a repository".
MUST NOT: a method whose only caller is a test.

If a method has no caller in `modules/` or `workers/`, delete it.

Name the method after the question (`get_open_for_user`, not `filter`). One question per method (01).

---

## Migrations

Alembic lives in `infra/db/migrations/`. How to write a revision: [07-migrations.md](07-migrations.md).

---

## Call chain (reminder)

```
modules/<capability>/ service
  → infra/db/repositories/<model>   (flush)
  → service commits
```

MUST NOT: router → repository
MUST NOT: worker → repository
MUST NOT: repository → another repository

---

## Done

- [ ] No `uow.py` / no sibling `engine.py`; runtime is `session.py`
- [ ] `http/deps` / worker use that scope; they do not commit
- [ ] Service still commits; repository still flushes
- [ ] No `commit()` in a repository
- [ ] No new unused CRUD method
- [ ] Relationships will not lazy-load after `await`
- [ ] Session still request- or job-scoped, `expire_on_commit=False`
- [ ] ORM does not outlive the session without a DTO
- [ ] Feature exceptions still raised in the service, not the repo
- [ ] A join lives in a read repository returning a DTO — not in the service, not in a write repo
- [ ] Uniqueness is a constraint and a transition is a conditional `UPDATE` — not a Python `if`
- [ ] Money is `Numeric`/`Decimal`, time is timezone-aware UTC, status is varchar + `CHECK`
- [ ] Engine sets a runtime `statement_timeout`
