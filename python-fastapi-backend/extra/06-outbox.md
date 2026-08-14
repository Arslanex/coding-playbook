# Extra 06 · Transactional outbox

WHEN: a write in Postgres **must not** commit without the matching queue/Kafka message existing, or the reverse (message without the row).
LOAD: this file **and** [06](../06-database.md), [08](../08-infra.md), [11](../11-workers.md). If [Extra 02](02-microservices.md) is on, load both. Not instead of 11.
SCOPE: one extra table + one worker kind so publish is not a second independent write. MUST NOT: `src/outbox/` as a backbone folder.

Default 11 is: service `commit()` then `infra/queue.publish`. If the process dies between those lines, the row exists and the job never ran. Accept that until this Extra is a shipped requirement (mail can retry from a UI; money/fan-out to another service often cannot).

---

## When to add

Add if losing the message (or sending it without the row) is a product defect.

MUST NOT: outbox "because microservices." One process, one DB, jobs that may be re-driven: stay on 11.
MUST NOT: two-phase commit across Postgres and the broker.

---

## How it attaches

No new city. The outbox is an `infra/db` table. Meaning of the event stays on the owning **module**.

```
infra/db/models/outbox.py          # table only
infra/db/repositories/outbox.py    # insert + list unpublished + mark published
modules/<capability>/service.py    # insert outbox row in the same session as the write
workers/jobs/publish_outbox.py     # read unpublished → infra/queue.publish → mark
```

Write owner of the **business** row is still `OrderService`. Outbox insert is that service's last step before `commit()`, not a second service that "knows all events" unless many modules share one poller **table** (one `Outbox` model, `aggregate_type` + payload — still filled by each module service).

MUST NOT: `modules/outbox/` as a product noun. Users do not GET `/v1/outbox`.
MUST NOT: repository `publish()`. Repos flush; the poller job calls `infra/queue`.

---

## One commit

```
OrderService.confirm
  → repos: update Order
  → repos: add OutboxMessage(topic, payload ids, created_at)
  → session.commit()          # row + outbox together
```

Worker:

```
list unpublished LIMIT N (16)
  → publish bytes (08, Extra 02 envelope)
  → mark published_at (or delete after success)
  → commit
```

MUST: the poller is idempotent. At-least-once to the broker (11). Consumer **service** still idempotent.

MUST: `SELECT … FOR UPDATE SKIP LOCKED` (or equivalent) so two poller pods do not send duplicates as a thundering herd. Duplicates that still happen: consumer handles them.

Payload: ids + envelope, frozen DTO on the owning module (11). MUST NOT: ORM in JSONB as the contract. MUST NOT: tokens, file bytes.

Failures: broker down → leave unpublished, retry (11 transient). Poison payload → DLQ row or `published_at` + error column; MUST NOT: block the poller on one bad row (skip + metric).

---

## [Extra 02](02-microservices.md)

Cross-service events use this Extra when the publisher's DB is the source of truth. Billing MUST NOT: read orders' outbox table. It consumes the **broker** topic.

Kafka vs Rabbit: [Extra 02](02-microservices.md). Outbox is how the **producer** reaches `infra/queue/`, not a second broker.

---

## Done

- [ ] Business write and outbox row share one `commit()`
- [ ] A worker publishes; modules do not call the broker in the request when this Extra is on
- [ ] Payload is ids + envelope; poller uses SKIP LOCKED + LIMIT
- [ ] No `src/outbox/`; no user-facing outbox API
