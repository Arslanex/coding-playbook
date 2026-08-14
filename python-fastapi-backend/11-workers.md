# 11 · Workers

WHEN: enqueueing work, adding a job file, the worker process, retry/DLQ, or "does this run in HTTP or out of band?".
LOAD: this file only.
RELATED: 08 (queue client) · 06 (session scope, idempotent writes) · 09 (the service the job calls) · 03 (retry policy, concurrency) — open only if the task is also that topic.
SCOPE: `src/workers/` and how it uses `infra/queue/` + `modules/*/service.py`.

A worker is a consumer. It reads a payload, calls a module service, exits. Stateless. It does not know HTTP.

---

## When this is a job

Stop at the first yes. The **rule** still lives on the module service. The job is only the trigger.

1. Must not run inside the HTTP request (slow, retryable, fan-out, vendor I/O: mail, webhook, export, large backfill)?
   → enqueue from the **service** (after `commit()` of the request use case, or as the use case). Handle in `workers/jobs/`.
2. Client must wait for the result in the same response (create order, cancel)?
   → **not a job** — stay on the request path.
3. A cron/tick with no HTTP?
   → still a job. Scheduler publishes; the handler is the same. MUST NOT: put the rule in the scheduler.

MUST NOT: a job because "the router should stay thin" — the router is already thin (09, 10).
MUST NOT: a job that is the only place the rule exists. Write `OrderService.issue_invoice` first; the job calls it.
MUST NOT: HTTP handler `BackgroundTasks` as a substitute for the queue. Process death loses the work.

Until the first enqueue exists: do not create `workers/`. First job = `runner.py` + `jobs/<kind>.py` + `infra/queue/` client (08) together.

---

## Files

```
src/workers/
  runner.py              # process entry. Not main.py.
  jobs/
    issue_invoice.py     # one job kind. Name = the work.
```

`runner.py` — `configure_logging()` (04). Open infra pools (same as API lifespan: engine, queue consumer, cache if needed). Consume loop. Dispatch `job_type` → handler. Classify retry vs DLQ. Close pools on shutdown. MUST NOT: a route. MUST NOT: SQL. MUST NOT: a product `if order.status`.

`jobs/<kind>.py` — parse **this** payload, open one session scope (06), `OrderService(session)` / `MailService(session)`, `await service.…()`, return. Logger: `worker` (04). MUST NOT: `commit()` (service does). MUST NOT: repository. MUST NOT: `http/`. MUST NOT: call another job file. MUST NOT: SendGrid / SMTP / S3 SDK — those clients are `infra/` (08).

Same noun, not this folder: enqueue + rule → `modules/<capability>/service.py`. Bytes on the wire → `infra/queue/`. Table → `infra/db/`.

Tests: `tests/workers/` mirrors. Service behavior is still tested under `tests/modules/`.

One runner, many job files. MUST NOT: one OS process per kind on day one. Split processes later only when kinds must autoscale independently — same files, different consume filter (`job_type` / topic from `config/`).

MUST NOT: `workers/services/`, `workers/utils.py`, `notification_worker.py` as a bag of unrelated kinds. File = one `job_type`.

---

## Runner loop

```
start
  configure_logging
  open pools (session factory, queue consumer)
  until shutdown:
    msg = consume()                          # infra/queue
    bind request_id=job_id, user_id          # 04, reset in finally
    handler = dispatch[msg.job_type]
    async with session_scope:                # 06, one job = one session
      await handler(msg, session)
    ack
  close pools
```

Dispatch is a dict in `runner.py` (or a tiny registry next to it when 01 splits). Unknown `job_type` → permanent fail → DLQ. MUST NOT: a plugin loader.

Concurrency: a semaphore from `config/` (how many jobs in this process). Must fit this process's DB pool ([16-performance.md](16-performance.md)). Each in-flight job has its **own** session. MUST NOT: one process-global session. MUST NOT: pass an ORM from job A to job B.

The HTTP app (`main.py`) and the worker (`workers/runner.py`) are two processes, one codebase. MUST NOT: run the consume loop inside FastAPI lifespan.

---

## Payload

Envelope (every job): `job_id`, `job_type`, `user_id` (nullable if the system enqueued it), `request_id` (correlates to the HTTP request that enqueued; else = `job_id`), `enqueued_at`.

Body: **ids and scalars** (`order_id`, `storage_key`). Frozen Pydantic / dataclass on the **owning module** (09: this is a DTO — open `dto.py` when the worker imports it). The worker imports that type. The module MUST NOT import `workers/`.

MUST NOT: ORM instance, `AsyncSession`, file bytes, password, token, raw `dict` with no schema.
MUST NOT: require `tenant_id` unless the product has tenants ([Extra 01](extra/01-multi-tenant.md)). Logs follow the same rule (04).
New fields: defaulted, so old messages still parse.

Enqueue:

```
OrderService.confirm  →  commit  →  infra/queue.publish(job_type, payload)
```

Publish after the write commit so a crash does not process a job for a row that never landed. MUST NOT: publish from `router.py`. MUST NOT: publish from a repository. If a crash *after* commit *before* publish is also a defect: [Extra 06](extra/06-outbox.md) (outbox), not a second `publish` in the router.

If the client must poll: router returns `202` + `job_id` because the **service** returned that id. Job status that users see is a product row or object in storage — not a required Redis `job:state` on day one. Add a jobs capability (09) only if the product has a job resource.

---

## What the job file does

```
parse payload (module DTO)
  → OrderService(session)
  → await service.issue_invoice(order_id, …)
```

Idempotency lives on the **service** (queue delivers at least once). Two runs must not send two mails or insert two rows. Worker MUST NOT: "just send". If two jobs write the same resource, the **service** takes `infra/cache` lock; the job file does not invent lock keys.

Workers do not call workers. Fan-out = service publishes another message. Job A MUST NOT: `import jobs.b`.

---

## Vendor client (mail)

The mailing **worker** has no mail client. SendGrid/SMTP is a vendor — `infra/email/` (08). Open that folder when the first send exists, not on day one of empty `infra/`.

```
modules/orders/service.py          # cancel commits, then queue.publish("send_mail", {order_id})
modules/mail/service.py            # to / template / already-sent. Calls infra/email.
infra/email/client.py              # PUT to SendGrid. No "order cancelled" sentence.
workers/jobs/send_mail.py          # payload → MailService.send_for_order(order_id)
infra/queue/                       # bytes only
```

`MailService` is the owner of mail meaning (09: two modules need send → this package, others call the service). If **only** orders ever send, skip `modules/mail/` — `OrderService` calls `infra/email/` directly; the job still calls `OrderService`, not the client.

MUST NOT: `workers/jobs/sendgrid.py`, `workers/mail_client.py`, `modules/mail/sendgrid.py`.
MUST NOT: `workers/` import `infra/email` (or `infra/storage`, `infra/payments`). Only `infra/queue` (consume) and `infra/db/session.py` (scope).

Same split for export (storage client in `infra/storage/`, job calls `ExportService`) and webhooks (HTTP client in `infra/` under the capability folder, job calls the module).

---

## Retry and DLQ

`infra/queue/` ack / nack / retry / DLQ. Policy per `job_type` in `config/` (attempts, backoff). MUST NOT: magic numbers in the job file.

Runner decides from the **exception type** (05). MUST NOT: `except Exception` in `jobs/`. MUST NOT: map to HTTP JSON.

- `ServiceUnavailableError`, timeout, vendor 5xx / 429 → transient → nack/retry (honor `Retry-After` when present).
- Payload validation, `AuthenticationError`/`AuthorizationError`, feature `NotFoundError` (row gone while queued), other 4xx-class feature errors → permanent → DLQ, `get_logger(LoggerName.WORKER).error(..., exc_info=True)` (04). MUST NOT: `LoggerName.ERROR`.
- Attempts exhausted → DLQ, same `WORKER` error log. MUST NOT: drop the message.

DLQ depth is an operational metric. A silent DLQ is lost work.

---

## Relation to other layers

```
modules/*/service.py
  → infra/queue.publish          # enqueue
  → infra/db (commit)

workers/runner.py
  → infra/queue.consume
  → workers/jobs/<kind>.py
       → modules/*/service.py    # same facade as HTTP
            → repos / infra clients
            → commit
```

`workers/` → may import: module `service` + `dto`, `infra/queue` (consume), `infra/db/session.py` (scope), `config/`, `shared/logging`, `shared/errors` (classify).

`workers/` → MUST NOT import: `http/`, `infra/db/repositories`, `infra/db/models`, `infra/email` (or other vendor clients), module `router` / `schemas` / helpers.

`http/` → MUST NOT import `workers/` (10).
`infra/` → MUST NOT import `workers/`.
Modules → MUST NOT import `workers/`. They publish through `infra/queue`.

---

## Done

- [ ] Rule is on a module service; job file only parses, opens session, calls, returns
- [ ] Module does not import `workers/`; worker does not import `http/` or repositories
- [ ] Payload is ids + envelope, typed on the owning module; no ORM, no secrets
- [ ] One session per job; service commits; runner acks
- [ ] At-least-once: service is idempotent
- [ ] Transient vs permanent; exhausted → DLQ + `LoggerName.WORKER` ERROR `exc_info=True`
- [ ] Vendor SDK (SendGrid, S3, …) is under `infra/<capability>/`, not in `workers/` or the job file
