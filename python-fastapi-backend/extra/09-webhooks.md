# Extra 09 · Webhooks (as a product)

WHEN: **customers** register HTTPS endpoints and this backend **delivers** events to them (or you ingest many partner webhooks as a platform), not a single Stripe callback.
LOAD: this file **and** [08](../08-infra.md), [09](../09-modules.md), [11](../11-workers.md), [15](../15-security.md). Not instead of them.
SCOPE: subscriptions + delivery. 15 still covers **one** inbound vendor HMAC. MUST NOT: `src/webhooks/` as a backbone. MUST NOT: treat 15's inbound snippet as this Extra.

If the only webhook is "Stripe POSTs to us": stay on 15 + `modules/billing/` (or payments) + `infra/payments/`. Do not open this Extra.

---

## Two directions

**Outbound (this Extra's default)** — we call the customer's URL.

**Inbound platform** — many providers POST to us. Same module pattern; verify **before** the service (15); then persist and enqueue. MUST NOT: one `webhooks.py` that both signs outbound and parses every vendor.

---

## Where it sits

```
modules/webhooks/          # Endpoint CRUD, event types, sign, delivery status
infra/db                   # WebhookEndpoint, WebhookDelivery
infra/http or infra/webhooks_dispatch/   # POST bytes, timeouts (16). No "order cancelled" sentence.
workers/jobs/deliver_webhook.py
```

Event **meaning** stays on the owning noun (`OrderService.confirm` decides `order.confirmed`). It calls `WebhookService.enqueue(event_type, payload_ids)` after commit (11) or [Extra 06](06-outbox.md).

MUST NOT: `OrderService` HTTP-POST the customer. MUST NOT: vendor SDK in the job file (08, 11).

---

## Models

- `WebhookEndpoint` — `id`, `user_id` (or [Extra 01](01-multi-tenant.md) `tenant_id`), `url`, `secret_hash` or encrypted secret, `event_types`, `disabled_at`. URL must be HTTPS. MUST NOT: store the signing secret in logs (04, 05).
- `WebhookDelivery` — `id`, `endpoint_id`, `event_type`, `payload` JSONB (ids + public fields, no tokens), `status`, `attempt`, `next_attempt_at`, response code.

Write owner: `WebhookService`. Authz: owner or tenant membership; same 404 (15).

Customer APIs (12):

```
POST/GET/PATCH/DELETE /v1/webhook-endpoints
GET /v1/webhook-deliveries?endpoint_id=   # cursor list
POST /v1/webhook-endpoints/{id}/rotate-secret
```

---

## Delivery

```
commit source + enqueue (or outbox)
worker: load due deliveries
  → sign body (HMAC, timestamp in a header)
  → POST with timeout from config/ (16)
  → 2xx → succeeded
  → timeout / 5xx / 429 → retry backoff (11), honor Retry-After
  → 4xx (except 429) → permanent fail, stop
```

MUST: at-least-once. Receiver may see duplicates — include `delivery_id` / `event_id` in the body so they can ignore. MUST NOT: "just POST" without a delivery row.

SSRF: MUST NOT: deliver to link-local, metadata IPs, or non-HTTPS. Resolve and check before connect (helper on `WebhookService` or dispatch client). MUST NOT: follow redirects to a private IP.

Disable the endpoint after N permanent failures (`config/`). `audit` on create, rotate, disable (04, 15).

Inbound (when this Extra includes it): path `/v1/webhooks/inbound/{provider}` (or a module per provider). HMAC + timestamp **before** `*Service`. Then 200 fast + job (11). MUST NOT: 500 on a valid signature because the job failed — persist first.

---

## Done

- [ ] Endpoints and deliveries are a module; HTTP POST is infra; job delivers
- [ ] Source service enqueues after commit (or [Extra 06](06-outbox.md)); it does not POST
- [ ] HMAC + timestamp; HTTPS; SSRF checks; retries on the delivery row
- [ ] Secrets not logged; rotate exists; same 404 on other people's endpoints
