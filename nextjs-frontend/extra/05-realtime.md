# Extra 05 · Realtime (UI)

WHEN: the screen must update without polling (status ticks, presence).
LOAD: this file **and** 05, 07, 12, 15. Backend: [python-fastapi-backend/extra/05-realtime.md](../../python-fastapi-backend/extra/05-realtime.md).
SCOPE: the island that listens. Source of truth stays FastAPI + first server render (07).

---

## Decide

SSE if server → client only. WebSocket if both directions. MUST NOT: WebSocket because it is fashionable.

A **client leaf** opens the stream (05). The page still SSR the current resource (07). The stream patches or calls `router.refresh()`.

MUST NOT [critical]: `?token=` on the socket URL — query strings land in proxy and access logs ([backend 15-security](../../python-fastapi-backend/15-security.md)). Cookie or first-message auth as the API Extra says.

MUST NOT: a module global of sockets as the product store (13). MUST NOT [critical]: skip 404/authz because a frame arrived.

Disconnect: show a quiet state, reconnect with backoff from `lib/env.ts`. MUST NOT: a spinner that eats the last good SSR.

---

## Done

- [ ] First paint still server data
- [ ] Stream is a client leaf; no token in the query string
- [ ] Refresh or patch; not RAM as truth
