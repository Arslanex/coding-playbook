# Extra 05 · Realtime (WebSocket / SSE)

WHEN: the client must receive events **without polling** (live order status, typing, presence).
LOAD: this file **and** [10](../10-http.md), [11](../11-workers.md), [13](../13-identity-security.md), [15](../15-security.md), [16](../16-performance.md). Not instead of them.
SCOPE: a second transport next to `http/`. Same module services. MUST NOT: `src/realtime/` as a product city. MUST NOT: scaffold `src/ws/` because this file exists — only when the first socket ships.

Default remains request/response (12). Polling or `202` + GET is enough until a live stream is a shipped requirement.

---

## SSE vs WebSocket

Stop at the first yes.

1. Server → client only (status ticks, one-way log)?
   → **SSE** on the **module** router (`StreamingResponse`). Still HTTP. No `ws/` folder.
2. Both directions (client sends, server pushes)?
   → **WebSocket**. Shell: `src/ws/` next to `http/` (same idea as `rpc/` in 13).

MUST NOT: WebSocket because the frontend "likes sockets." MUST NOT: GraphQL subscriptions as the first realtime (13).

---

## Where it sits

```
src/ws/                 # Extra 05 WebSocket only
  server.py             # accept, heartbeat, close. No OrderService rules.
  deps.py               # identity — reuse http/deps CurrentUser if the same process
modules/<capability>/   # "may this user subscribe to order {id}"
infra/cache/            # pub/sub bytes (08). Channel name from the module.
workers/jobs/           # after the write: service publishes the event (11 or Extra 06)
```

`main.py` (or a second process entry) mounts the WS app and opens the same pools. [Extra 02](02-microservices.md) only if the socket farm must scale apart from HTTP.

SSE: no `ws/`. Route lives on `modules/<capability>/router.py`. Auth: same `Depends(get_current_user)`.

MUST NOT: `modules/realtime/`. MUST NOT: `http/sockets.py` that knows orders.

---

## Identity

Same actor as HTTP (13, 15). Protected stream: no anonymous default.

Browser: cookie session / refresh already chosen in 13. MUST NOT: `?token=` on the WS URL (15, 13).

First-message Bearer is allowed when there is no cookie. MUST NOT: a second JWT library. `ws/deps` verifies; it does not login (`modules/auth/` still issues).

Subscribe is **authz**: `OrderService.assert_visible(order_id, user_id)` before the socket is bound to that id. Missing and not-owned → same 404 close (or a close code that does not confirm the id). MUST NOT: trust a client-sent `user_id`.

Rate-limit connects (10). Caps: open sockets per user and per process from `config/` (16).

---

## Fan-out (not RAM)

The socket object lives in **this** process. The **event** must not.

MUST NOT: `dict[user_id, WebSocket]` (or a module global) as the only way another request/job notifies the client. A second pod will not see it (02).

```
OrderService.cancel
  → commit
  → publish "order.cancelled" (queue or Redis pub/sub — module chooses the channel)
each process
  → subscription task
  → send to sockets this process holds for that order_id
```

Redis pub/sub is reconstructible fan-out (13): losing it means a missed tick, not a wrong order. Source of truth stays Postgres. Worker/API that made the write still goes through the **service**.

Heartbeat / idle timeout from `config/`. MUST NOT: hold a DB session for the life of the socket (16). Open a session only for subscribe-authz and then close.

Payload to the client: the same resource fields as HTTP schemas (12), or a small event DTO on the module. MUST NOT: ORM. MUST NOT: stack/DSN.

---

## Done

- [ ] SSE on the module router, or `ws/` shell — not `src/realtime/`
- [ ] Authz on subscribe; no `?token=`
- [ ] Fan-out via cache/queue; no process-local dict as the bus
- [ ] No DB session held for the connection lifetime
- [ ] Same 404/audit as HTTP for the underlying resource
