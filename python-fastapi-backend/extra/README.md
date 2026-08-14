# Extra (optional)

WHEN: the product **already** needs one of the shapes below.
LOAD: only the matching file. Do not load Extra with 01–16 on a single-process API. Stack map: [../README.md](../README.md).

MUST: 01–16 still apply. Extra **adds**. It does not replace `http/` · `modules/` · `infra/` · `workers/`.
MUST NOT: create `src/extra/`, `src/microservices/`, `src/agents/`, `src/ws/`, `src/outbox/`, `src/sso/`, `src/search/`, `src/webhooks/`, `src/gdpr/` because this folder exists.

## LOAD

1. Several customers isolated — [01-multi-tenant.md](01-multi-tenant.md)
2. More than one microservice (one repo or many) — [02-microservices.md](02-microservices.md)
3. One agent or an agent team in the product — [03-agent-teams.md](03-agent-teams.md)
4. A library/package you wrote (execution, agent kit, ML, …) — [04-packages.md](04-packages.md)
5. Live WebSocket / SSE — [05-realtime.md](05-realtime.md)
6. Write and broker message must share a commit — [06-outbox.md](06-outbox.md)
7. Sign-in via an IdP (OIDC / SAML) — [07-sso.md](07-sso.md)
8. Ranked / engine search — [08-search.md](08-search.md)
9. Customers register webhook endpoints — [09-webhooks.md](09-webhooks.md)
10. Hide, retain, or erase rows — [10-retention.md](10-retention.md)
