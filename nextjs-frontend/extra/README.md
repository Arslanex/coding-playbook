# Extra (optional)

WHEN: the product **already** needs one of the shapes below.
LOAD: [AGENTS.md](../../AGENTS.md) and [agents/01-boundary.md](../../agents/01-boundary.md) if Extra gate unclear. Then only the matching file below — not all Extra.

MUST: `01`–`16` still apply. Extra **adds**. It does not replace `app/` · `features/` · `ui/` · `lib/`.
MUST NOT: create `frontend/extra/`, `app/i18n/` as a city, or backbone folders because this directory exists.

---

## WHEN → which Extra file

WHEN: tenant slug or host in the signed-in URL **already**.
LOAD: [01-multi-tenant.md](01-multi-tenant.md).

WHEN: two or more deployable Next apps **already**.
LOAD: [02-apps.md](02-apps.md).

WHEN: UI copy in more than one language **already**.
LOAD: [03-i18n.md](03-i18n.md).

WHEN: IdP sign-in in the browser **already**.
LOAD: [04-sso.md](04-sso.md).

WHEN: live SSE / WebSocket UI **already**.
LOAD: [05-realtime.md](05-realtime.md).

WHEN: search results screen with engine/ranking **already**.
LOAD: [06-search.md](06-search.md).

WHEN: file upload UX **already**.
LOAD: [07-uploads.md](07-uploads.md).

WHEN: in-product agent run transcript UI **already** — not the IDE coding agent.
LOAD: [08-agents.md](08-agents.md).

Stack map: [../README.md](../README.md). Agent routing: [../../agents/README.md](../../agents/README.md).
