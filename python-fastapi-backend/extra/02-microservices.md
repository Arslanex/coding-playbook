# Extra 02 · Microservices

WHEN: two or more **deployable** backends (each with its own process and scale), in one repo or in many.
LOAD: this file **and** [02](../02-file-structure.md), [08](../08-infra.md), [09](../09-modules.md), [11](../11-workers.md), [12](../12-api.md). Not instead of them.
SCOPE: tree changes and how services talk (HTTP + Kafka / RabbitMQ). A second **module** in the same process is not this Extra.

Default remains one `backend/` (02). Split only when at least one of these is true — and it is true **today**, not on a roadmap:

- **Deploy** — the two halves must ship on independent schedules, with independent rollback.
- **Scale** — one half needs a different machine shape (GPU, memory) or scales on a different signal.
- **Failure** — one half must keep serving while the other is down, and that isolation is a product promise.
- **Trust** — the two halves have different blast radius or compliance scope (card data, a customer-run sandbox).

This file assumes that decision is already yes. MUST NOT: split because a query is slow — that is [16-performance.md](../16-performance.md). MUST NOT: split because the codebase "feels big" — that is 01 and 09.

Each service **is** the playbook tree: `http/` · `modules/` · `infra/` · `workers/`. MUST NOT: a service that is only routers or only a broker consumer.

---

## One repo, several services

The repo root is no longer a single `backend/src`. Each service is a **full** backbone, not a layer slice.

```
/
├── services/
│   ├── orders/                 # deployable
│   │   ├── src/
│   │   │   ├── main.py
│   │   │   ├── config/
│   │   │   ├── shared/         # errors + logging only (02) — copy or workspace pkg
│   │   │   ├── http/
│   │   │   ├── infra/          # this service's db/cache/queue client
│   │   │   ├── modules/        # only this service's capabilities
│   │   │   └── workers/
│   │   └── tests/              # mirror of this service
│   ├── billing/
│   │   ├── src/ …              # same shape
│   │   └── tests/
│   └── identity/               # signs JWT (13). others only verify.
├── packages/                   # Extra 04 — optional shared *libraries*
│   └── events/                 # pydantic envelopes. MUST NOT: ORM models.
└── deploy/
```

What **changes** vs one backend:

- CI/deploy is **per** `services/<name>/` (path filter). One service's test run does not boot the others.
- Each service that owns tables has **its own** `infra/db` and **its own** `migrations/` beside that service's `src/` (02, 07). MUST NOT: one `migrations/` for all services. MUST NOT: a shared chain at the repo root that several services replay.
- `modules/` inside a service holds only that service's nouns. Billing does not contain `modules/orders/`.
- Cross-service types: event/HTTP DTOs in `packages/events/` (or duplicated frozen schemas until the second consumer). MUST NOT: `packages/models/` of SQLAlchemy.
- `shared/errors` parents: one small package **or** a copy. Feature errors stay in that service's module (05).

MUST NOT: `services/orders/src/api` + `services/orders/src/domain` as a return to three cities.
MUST NOT: a `services/libs/utils`.

Write ownership across services: only `services/orders` writes `orders`. Billing calls orders **HTTP** or consumes an **event**. MUST NOT: billing's repo `UPDATE` the orders table.

---

## Separate repos, same tree

Each repo is `services/orders/` above, as its own git root (still `src/http|modules|infra|workers`).

What stays identical: 01–16, URL prefix `/v1/…` on **that** service's resources, error JSON (12), worker payload envelope (11).

What is not shared as source: ORM, Settings class, `http/deps` internals. Identity: one issuer repo; others verify with the **published** JWT key (13).

Discover: `config/` base URLs (`ORDERS_BASE_URL`, `KAFKA_BROKERS`, `AMQP_URL`). MUST NOT: hardcode hosts.

---

## How they talk

Two doors. Pick per **use case**, not per company fashion.

1. **HTTP** — the caller needs an answer now (quote, "does this order exist"). Caller's **service** uses an HTTP client in `infra/` (capability folder, e.g. `infra/orders_client/` or a generic `infra/http/` **only** if it has no product noun — prefer a client named after the **other service's capability**). Timeouts, retries as transient (05 `ServiceUnavailableError`). Same `{error_code, message, details}`. MUST NOT: router-to-router. MUST NOT: open the other service's DB.

2. **Queue** — the caller must not wait (invoice after confirm, mail, fan-out). Publisher: owning **service** after `commit()` (11). If losing that publish is a defect: [06-outbox.md](06-outbox.md) (row + outbox in one commit; a worker hits the broker). Consumer: the other service's `workers/jobs/`. Payload: ids + envelope (`job_id`, `job_type` / `event_name`, `user_id`, `request_id`, `enqueued_at`). MUST NOT: ORM, file bytes, tokens.

MUST NOT: service A import service B's Python package to "just call OrderService" across a repo boundary — that is still one process pretending.

Idempotency: at-least-once on both Kafka and Rabbit. Consumer **service** is idempotent (11).

---

## The edge (outside → in)

The two doors above are service **to** service. This is how the outside gets in, and it is the first thing a third service forces you to decide.

### One public origin

MUST: the browser and the Next server know **one** address. Path prefixes route behind it — `/v1/orders` → orders, `/v1/invoices` → billing.

MUST NOT: a public URL per service. The frontend keeps one `API_BASE_URL` (frontend 04, 09) and one wrapper. Your service topology is not the browser's business, and splitting a service later must not become a frontend change.
MUST NOT: `NEXT_PUBLIC_ORDERS_URL` + `NEXT_PUBLIC_BILLING_URL`. That publishes the topology to every visitor and breaks frontend 04's rule against a second public env var.

SHOULD: put the UI and the API on the **same** origin (`app.example.com` and `app.example.com/v1/…`). Then there is no CORS at all, and no preflight on every write.

### The edge is platform config, not a service

MUST NOT: a `gateway/` FastAPI or Express process under `services/`. This file already says a service must be the full backbone — `MUST NOT: a service that is only routers`. A routing-only process is exactly that, plus a deploy unit and an on-call page.

WHERE: `deploy/` at the repo root — ingress rules, load balancer or CDN config, or an nginx/Envoy file. Whatever the host already speaks. It holds routing, TLS, and the coarse limit; **not** per-service build files, which stay beside each service (02 *Build and release*).
MUST NOT: business rules at the edge. No "cancel if unpaid", no response rewriting, no auth decisions.
MUST NOT: write these files unasked — deploy config is the user's ([agents/01-boundary.md](../../agents/01-boundary.md)).

### What moves there, what stays

At the edge, **once**:

- TLS termination (15 already says "at the platform")
- Path routing to services, and the public DNS name
- A coarse anonymous rate limit. Per-service limits **multiply**: three services at 100/min is 300/min to one attacker with no rule broken.
- `X-Request-ID` generated when the client sent none, so every service shares one id (below)
- CORS, **if** the UI is on a different origin

In each service, **still**:

- Identity — each service verifies the JWT with the published key (13). MUST NOT: the edge validates and forwards claims as trusted headers. A request that reaches the service another way must fail exactly the same.
- Authz — the owning service checks ownership (15). The edge never decides who may cancel an order.
- Named-surface rate limits — login, password reset, OTP (15 layer 3). Those are product limits and they belong with the module that owns them; the edge's coarse limit does not replace them.
- Idempotency — transport, but the outcome is stored in **that** service's cache (10).
- `/docs` off in production — per service, because a service reachable directly must not expose it either.

MUST: exactly one place holds the allowed origins. If an edge fronts every browser call, that place is the edge and the services drop their CORS middleware (10 middleware order, item 2). MUST NOT: keep both "just in case" — two lists drift, and the stale one is found by a broken deploy.

### The edge is a convenience, not a control

MUST: everything the edge does must still be safe when the edge is bypassed. A pod is reachable inside the cluster; a misrouted rule sends traffic straight at it; a developer curls it directly.

MUST NOT: move a control to the edge and delete it from the service. Moving CORS is fine — CORS is a browser mechanism and a non-browser caller was never protected by it. Moving **identity or authz** is not.

A hidden button is not a control, and neither is a routing rule. Curl is.

---

## The trace must survive the hop

One backend has one `request_id` per request (10). With several, a single user action becomes N log streams — and unless the id crosses each hop, they cannot be joined. A failure in `billing` caused by an `orders` call is then two unrelated incidents.

MUST: the HTTP client in `infra/<other>_client/` **forwards** the caller's `X-Request-ID` on every outbound call. It does not generate a new one.
MUST: the receiving service's `request_id` middleware accepts an incoming `X-Request-ID` rather than always minting one (10) — that rule already exists per service; it is what makes forwarding work.
MUST: the queue envelope already carries `request_id` (above). The consumer **binds** it to its own log context before the job runs (04), so worker lines join the same trace.

MUST NOT: a second correlation header alongside `X-Request-ID`. One id, one name, every service.
MUST NOT: let the id die at the browser boundary — the frontend already sends it (frontend 09) and reports it on failures (frontend 06). That is the same value end to end.

WHEN: a call fans out to several services or several jobs.
HOW: `request_id` stays the same for the whole fan-out; per-hop identity is the service name in the log line, not a new id.

MUST NOT: build distributed tracing (spans, a collector, a backend) before this. A forwarded id and structured logs answer "what else happened in this request" for a long time. Tracing is a later decision with its own cost.

---

## Kafka vs RabbitMQ

The folder is still `infra/queue/` (08). Vendor is a **file** inside (`kafka.py` / `rabbitmq.py`). MUST NOT: `infra/kafka/` + `infra/rabbitmq/` as two capability folders. Two brokers only if they do **different jobs** (log vs task) — then still one `queue/` with two clients, or Extra 02 later if they must scale apart.

**RabbitMQ** — work queue. One consumer group competing on a queue. Ack/nack, DLQ, routing key, prefetch. Use for: "do this job" (`issue_invoice`, `send_mail`). Default for 11-style jobs.

**Kafka** — event log. Topic + partition key (usually aggregate id, `order_id`) so one aggregate stays ordered. Consumer **group** = one service type. Replay by offset. Use for: many independent consumers of the same fact (`order_confirmed`). MUST NOT: use Kafka as a private RPC to one worker if Rabbit already does jobs — two semantics, two bugs.

Rules for **both**:

- Topic/queue **name** from `config/` + owning module (`orders.order_confirmed.v1`). MUST NOT: infra invent `order:cancelled:v2` as a business event (08).
- Compatibility: new fields defaulted (11). Bump `.v2` only on a breaking change; run both until consumers move.
- Key (Kafka): aggregate id. Rabbit routing key: same id or capability. MUST NOT: random key if you need per-order order.
- Failures: transient → retry; permanent → DLQ (11). MUST NOT: `except Exception` in the job.
- One `job_type` / event is not published to **both** brokers.

---

## Done

- [ ] Each deployable has the full backbone, not a layer slice
- [ ] One writer per table/aggregate; no shared ORM package
- [ ] Sync = HTTP from service; async = queue after commit (or [Extra 06](06-outbox.md) outbox)
- [ ] `infra/queue/` named by capability; Kafka vs Rabbit chosen by log vs task
- [ ] Consumer is idempotent; payload is ids + envelope
- [ ] `X-Request-ID` forwarded on every service-to-service call and bound by every consumer
- [ ] One public origin; frontend still has one `API_BASE_URL`; no `gateway/` service
- [ ] Identity and authz still in each service; only CORS/TLS/coarse limits moved to the edge
