# Extra 10 · Retention, soft-delete, erasure

WHEN: rows must be **hidden** and later restored, **kept** for a legal period, or **erased/anonymised** on request (GDPR-style) — and that is a shipped requirement.
LOAD: this file **and** [06](../06-database.md), [07](../07-migrations.md), [09](../09-modules.md), [11](../11-workers.md), [15](../15-security.md). [Extra 08](08-search.md) if an index must drop the document. Not instead of 06.
SCOPE: columns, repo filters, jobs. MUST NOT: `SoftDeleteMixin` on every table (06). MUST NOT: `src/gdpr/` as a backbone.

Default 06: delete means `DELETE` (or a real status on that noun, `cancelled`, which is **not** this Extra). Do not sprinkle `deleted_at` "for safety."

---

## Decide per table

Stop at the first yes. Different tables may pick different answers.

1. User must undo a hide (trash / restore)?
   → **Soft-delete**: `deleted_at` timestamptz null = live. Restore is a service verb.
2. Law requires keep-then-destroy (invoices, logs)?
   → **Retain**: `retain_until` (or a status + job). After that, hard delete or anonymise — product rule on the **service**.
3. Person requests erase, and you must not be able to read them back?
   → **Erase**: job that deletes or replaces PII with tombstones. `audit` the request, not the PII (04, 15).
4. None of the above?
   → **not this Extra** for that table.

MUST NOT: one mixin that does all three. MUST NOT: `deleted_at` on `User` as a substitute for 13 logout.

---

## Soft-delete (hide)

When this Extra is on for a table:

- Column `deleted_at` nullable. Index live rows (`WHERE deleted_at IS NULL`) for the queries that list (07, 16).
- Unique email/slug: **partial** unique index on live rows. MUST NOT: unique `(email)` that blocks reuse after hide unless the product forbids reuse.
- Repository **list/get** used by the product default: `deleted_at IS NULL`. A `get_including_deleted` only when restore/admin needs it (06: add when called).
- Service: `delete` sets `deleted_at`; `restore` clears it. Authz same as write (15). Missing, not-owned, **and already hidden** on a public GET → same 404.

MUST NOT: mixin methods `soft_delete()`. The service sets the column (or a typed repo method `mark_deleted(id, now)` with no business `if`).

Workers: payload id may point at a hidden row — the **service** decides (no-op vs error). MUST NOT: skip authz because it came from the queue (11, 15).

---

## Erase vs anonymise

Erase job (11), never the HTTP request (16):

- Load the graph the **module** owns (user → sessions → …). Other aggregates: [Extra 02](02-microservices.md) HTTP/events, or this service if one DB.
- Anonymise when a row must remain (invoice law): strip email/name, keep `id` + money fields. Hard delete when nothing must remain.
- [Extra 08](08-search.md): delete or rewrite the search document in the same use case (job steps, [Extra 06](06-outbox.md) if needed).
- [Extra 05](05-realtime.md): do not rely on RAM; next event simply has no row.
- Redis: delete reconstructible keys you know (13). Accept that a flushed cache is fine; accept that a live JWT may work until `exp` unless `jti` denylist (13).

MUST NOT: `DELETE FROM users` in a migration (07). MUST NOT: log the old email on erase.

HTTP: `POST /v1/users/me/erasure-requests` → `202` + id (12). Status is a product row (`ErasureRequest`), write owner `UserService` / `AuthService`.

---

## Retention jobs

Scheduler publishes; handler is `workers/jobs/` calling the service (11). Batch `LIMIT` (16). Re-runnable. MUST NOT: one `UPDATE` of the whole table in Alembic (07).

---

## Done

- [ ] Each table chose hide, retain, erase, or none — no global mixin
- [ ] Live queries filter `deleted_at IS NULL`; unique indexes are partial when reuse is allowed
- [ ] Public GET still 404 for hidden/not-owned
- [ ] Erase/retain is a job; PII not in logs; search/cache/JWT follow this Extra + 13
