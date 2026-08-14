# 07 · Migrations

WHEN: adding, changing, or dropping a table, column, index, constraint, or seed row.
LOAD: this file **and** 06 — the revision follows a model change, and 06 decides the column type.
RELATED: 02 (`infra/db/migrations/` placement) · 16 (index vs query) — open only if the task is also that topic.
SCOPE: Alembic vs one PostgreSQL database. One revision chain.

A revision is schema history. It must run on a live app that is still on the previous code (rolling deploy).

---

## Folder

```
infra/db/migrations/
  env.py
  script.py.mako
  versions/
```

`env.py` imports `DeclarativeBase.metadata` from `models/base.py` and the model package so autogenerate sees tables. MUST NOT: put upgrade logic in `env.py`.

`versions/` — one file per revision. MUST NOT: a `helpers/` dump. A shared concurrent-index helper may live as one file next to `env.py` when a second revision would otherwise copy the same `autocommit_block`.

---

## How to add a revision

1. Change the ORM model (06).
2. Autogenerate.
3. **Read** the script. Alembic misses some type/index/constraint edits and invents others.
4. Keep one logical change per file (add nullable column **or** drop a column, not both).
5. Fill `downgrade()`. `pass` is forbidden. If irreversible, `downgrade` raises with why, and restore-from-backup is the path.

MUST NOT: merge autogenerate unread.
MUST NOT: edit a revision that already shipped. Write a new one.
MUST NOT: run ad-hoc SQL on production. The change is a revision.
MUST NOT: HTTP or other network calls inside `upgrade` / `downgrade`.
MUST NOT: import an ORM model class in a revision. Models change; old revisions must keep working. Use `op` / `sa` for DDL, `sa.table()` / `sa.column()` for data.

Files under `versions/` do not use the 01 file-header. They use the Alembic block:

```python
"""add external_ref to orders

Revision ID: …
Revises: …
Purpose: Nullable orders.external_ref for provider ids.
Backward compatible: yes — old code never reads this column.
"""
```

`Backward compatible: no` is only legal when expand/contract (below) already landed the safe half in an earlier release.

---

## Expand → contract

Old pods and new pods share the database during deploy. Destructive shape changes take two releases.

- **Add column** — nullable or with `server_default`. One revision is enough.
- **Drop column / table** — R1: remove all reads/writes in code and ship. R2: drop in a new revision.
- **Rename column or change type** — R1: add the new column, write both. R2: backfill, switch reads. R3: drop the old column. MUST NOT: in-place `ALTER TYPE` on a hot table.
- **New status value** — if the column is varchar + `CHECK` (06), this is dropping and re-adding one constraint. MUST NOT: a native PostgreSQL `ENUM`; `ALTER TYPE … ADD VALUE` cannot run in a transaction and locks the type. This is why 06 forbids native enums in the first place.
- **Add NOT NULL** — R1: backfill + `CHECK … NOT VALID`. R2: `VALIDATE CONSTRAINT`. R3: `SET NOT NULL`. Adding `NOT NULL` with a constant `server_default` on PG11+ skips a full rewrite for a **new** column.
- **Add FK / CHECK** — add `NOT VALID`, then `VALIDATE` (same revision is allowed only if validate is a separate statement after the add; prefer two revisions on large tables).

MUST NOT: add a column and drop another in the same revision.

---

## Locks (PostgreSQL)

Every `upgrade` starts with:

```python
op.execute("SET lock_timeout = '3s'")
op.execute("SET statement_timeout = '60s'")
```

If the lock cannot be taken, fail fast. Waiting forever queues all traffic.

Indexes on existing tables: `CONCURRENTLY` inside `autocommit_block()` — it cannot run in a transaction.

```python
with op.get_context().autocommit_block():
    op.create_index(
        "ix_orders_customer_id",
        "orders",
        ["customer_id"],
        postgresql_concurrently=True,
    )
```

---

## Data vs schema

MUST NOT: mix a destructive schema change and a large backfill in one revision.

Small, idempotent seed (roles, system rows): allowed in a revision. Fixed UUIDs. `ON CONFLICT DO NOTHING`. MUST NOT: `gen_random_uuid()` for seed PKs (IDs would differ per environment).

Large backfill: `workers/jobs/` calling a service ([11-workers.md](11-workers.md)). Batch it. Re-runnable. MUST NOT: one `UPDATE` of the whole table in a migration.

If a tiny data fix must live in Alembic, use `sa.table()` / `sa.column()`, batch `LIMIT`, stop when `rowcount == 0`.

---

## Runtime

App start does **not** run migrations. A migrate command (CI or release job) runs `alembic upgrade head` against the database, then pods roll.

MUST: new environments migrate to `head` before serving traffic.
MUST: log revision before/after (04: `system` or `audit` if the change is a production cutover). MUST NOT: log the DSN.

---

## Done

- [ ] One logical change; no add+drop in the same file
- [ ] Autogenerate output was edited by a human
- [ ] No ORM model import
- [ ] `downgrade()` filled or raises with a reason
- [ ] `lock_timeout` / `statement_timeout` set
- [ ] New indexes on live tables are `CONCURRENTLY`
- [ ] Old code can boot on this schema, or this is explicitly the contract half of expand/contract
- [ ] Shipped revisions were not edited
