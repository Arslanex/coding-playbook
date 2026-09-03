# 08 · Infra

WHEN: placing a client to Postgres, Redis, a queue, object storage, or a third-party HTTP API — or adding a new outside system.
LOAD: this file only.
RELATED: 06, 07 (if the system is Postgres) · 02 (placement) · 16 (this client's pool and timeout) · 03 (its settings) — open only if the task is also that topic.
SCOPE: `src/infra/` only.

Infra talks to systems whose **meaning we do not own**. Modules own the sentences ("cancel if unpaid"). Infra owns the verbs against a vendor ("SET this key", "PUT this object").

---

## When code belongs here

Stop at the first yes.

1. Deleting the vendor (Postgres, Redis, S3, Stripe, SendGrid) would delete this file, and **no product sentence would remain**?
   → `infra/`
2. The file is an ORM table, SQL for one model, or an Alembic revision?
   → `infra/db/` (06, 07)
3. The file is "cache this order 60s", "send mail when cancelled", "user may upload an invoice"?
   → **not infra** — `modules/<capability>/` ([09-modules.md](09-modules.md)). That module **calls** infra.

MUST NOT: HTTP routes, feature exceptions' JSON map, logger pipeline (`shared/logging/`), or `commit()` here.
MUST NOT: a second `utils/` under infra.

---

## Day-one folders

```
infra/
  db/         # 06, 07
  cache/
  queue/
  storage/
```

Pick the folder by **what the outside system is for**, not by the vendor name.

`infra/db/` — PostgreSQL. Session, models, repositories.
Put here: engine/session, tables, SQL.
Put elsewhere: "cannot cancel if paid" → module. The Alembic chain → `migrations/` beside `src/` (02, 07) — it is the one Postgres thing the app never imports. Details: 06, 07.

`infra/cache/` — key/value + TTL + lock primitives (Redis).
Put here: get, set, expire, acquire/release lock.
Put elsewhere: which key, which TTL, what to invalidate after cancel → module calls cache.
MUST NOT: encode order state machines in cache keys without the module choosing the key.

`infra/queue/` — publish/consume bytes (broker).
Put here: enqueue a payload to a topic/queue, ack/nack adapters.
Put elsewhere: what the job **means** → module. The process that pulls jobs → `workers/jobs/` ([11-workers.md](11-workers.md)). Workers import `infra/queue/` to consume; they still call a module service to do the work.

`infra/storage/` — object storage (S3-compatible).
Put here: put, get, delete, signed URL.
Put elsewhere: "user may upload an invoice" → module. MUST NOT: write user files to local disk (02).

Secrets and DSN construction used by these clients: `config/` + secret store. MUST NOT [critical]: hardcode credentials. MUST NOT: log DSN or bytes of objects (04, 05). Pool and timeout for each client: [16-performance.md](16-performance.md).

---

## When to open a new folder

Open `infra/<name>/` only when **a new class of outside system** appears — not a second use of an existing class.

Open:

- email SMTP/HTTP provider → `infra/email/`
  (the worker does not hold this client — [11-workers.md](11-workers.md))
- card payments → `infra/payments/`
- SMS → `infra/sms/`

Do not open:

- `infra/redis/` — that is `cache/`
- `infra/s3/` — that is `storage/`
- `infra/sqs/` / `infra/rabbitmq/` — that is `queue/`
- `infra/sendgrid/` — that is `email/` (vendor is a file inside)
- `infra/postgres/` — that is `db/`
- a folder because two modules call Redis — still `cache/`

Test: if you renamed the vendor next year, would the folder name become a lie? If yes, the folder is named wrong.

Until a second vendor exists, the folder holds **one client file** (plus tests under `tests/infra/…`). MUST NOT: `ports/` + `adapters/` for a single SendGrid client.

When a second vendor lands: add a port (protocol) in that folder and one file per vendor (`sendgrid.py`, `ses.py`). The module still depends on the port, not on SendGrid.

---

## Naming

Folder: **capability of the outside system**, English, singular, short.

- `db` `cache` `queue` `storage` `email` `payments`
- MUST NOT: vendor (`stripe`, `boto`, `aioredis`)
- MUST NOT: layer words (`clients`, `adapters`, `services`)

Files inside a folder: the adapter (`client.py` while there is one; vendor name when there are two).

Keys, topics, bucket names: come from `config/` or from the **module** that owns the meaning. Infra applies them; it does not invent `order:cancelled:v2` as a business event.

---

## Call direction

```
modules/<capability>/  →  infra/<folder>/     (client)
workers/jobs/          →  modules/…/service     (11)
workers/jobs/          →  infra/queue/        (consume only)
```

MUST NOT: infra import a module.
MUST NOT: `http/` import infra except `http/deps` opening `infra/db/session.py`, and `http/` reads for rate-limit, idempotency, and identity verification — the `jti` denylist and whether the account is still active (10, [13-identity-security.md](13-identity-security.md)). Those reach `infra/` whichever store backs them — which store is a durability question, answered in [13](13-identity-security.md), and it does not change what `http/` may import. MUST NOT: `http/` read any of them through `modules/*/service.py`. A narrow identity reader is not `infra/db/repositories/` and does not become importable by the rest of `http/`: it answers identity questions and nothing else, which is the difference from generic row access.
MUST NOT: infra/db repository import cache/queue/storage.

Failures talking to a vendor: raise `ServiceUnavailableError` (05). MUST NOT: leak vendor error bodies to the client.

---

## Done

- [ ] Vendor-delete test passed (no product sentence left in this file)
- [ ] Folder is a capability name, not a vendor name
- [ ] No new infra folder for a second Redis/S3/queue use
- [ ] Module still decides when/why; infra only how
- [ ] No `ports/adapters` tree for a single client
