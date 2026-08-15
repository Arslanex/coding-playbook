# Extra (optional)

WHEN: the product **already** needs one of the shapes below.
LOAD: [AGENTS.md](../../AGENTS.md) and [agents/01-boundary.md](../../agents/01-boundary.md) if Extra gate unclear. Then only the matching file below — not all Extra. Do not load Extra with all of `01`–`16` for a routine API change.

MUST: `01`–`16` still apply. Extra **adds**. It does not replace `http/` · `modules/` · `infra/` · `workers/`.
MUST NOT: create `src/extra/`, `src/microservices/`, `src/agents/`, `src/ws/`, `src/outbox/`, `src/sso/`, `src/search/`, `src/webhooks/`, `src/gdpr/` because this folder exists.

---

## WHEN → which Extra file

WHEN: two or more customers must not see each other's rows **and that is shipped**.
LOAD: [01-multi-tenant.md](01-multi-tenant.md).

WHEN: more than one deployable backend **already exists**.
LOAD: [02-microservices.md](02-microservices.md).

WHEN: the **product** runs in-product agent/team jobs — not the IDE coding agent.
LOAD: [03-agent-teams.md](03-agent-teams.md).

WHEN: this backend imports a library you wrote (execution kit, ML, …).
LOAD: [04-packages.md](04-packages.md).

WHEN: clients receive events without polling (WebSocket / SSE) **already**.
LOAD: [05-realtime.md](05-realtime.md).

WHEN: DB write and broker message must share one commit **already**.
LOAD: [06-outbox.md](06-outbox.md).

WHEN: users sign in through an IdP (OIDC / SAML) **already**.
LOAD: [07-sso.md](07-sso.md).

WHEN: ranked/engine search is shipped — not `LIKE` list only.
LOAD: [08-search.md](08-search.md).

WHEN: customers register HTTPS webhook endpoints **already**.
LOAD: [09-webhooks.md](09-webhooks.md).

WHEN: rows must be hidden, retained, or erased on request **already**.
LOAD: [10-retention.md](10-retention.md).

Stack map: [../README.md](../README.md). Agent routing: [../../agents/README.md](../../agents/README.md).
