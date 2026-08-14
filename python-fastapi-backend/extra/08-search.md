# Extra 08 · Search

WHEN: users **search** a corpus (orders, docs, catalog) and Postgres `LIKE` / a filtered list (12) is not enough — ranking, typo, facets, or a dedicated search engine is a shipped requirement.
LOAD: this file **and** [08](../08-infra.md), [09](../09-modules.md), [11](../11-workers.md), [12](../12-api.md), [15](../15-security.md). Not instead of them.
SCOPE: where the index lives and who decides what is searchable. MUST NOT: `src/search/` as a backbone. MUST NOT: `infra/elasticsearch/` as a vendor-named city (08).

Default: list endpoints with cursor + `WHERE` (12, 16). Add this Extra when that API cannot answer the query.

---

## Decide engine

Stop at the first yes.

1. One Postgres, simple ranking (`tsvector`, `pg_trgm`), same rows the module already owns?
   → Postgres FTS. Index in `infra/db` migrations (07). Query in that model's repository (or a `SearchRepository` that still does not steal writes). **No** `infra/search/` yet.
2. A search engine (OpenSearch, Typesense, Meilisearch) as an outside system?
   → `infra/search/` client (08). Documents are reconstructible: if the cluster is wiped, a job rebuilds from Postgres.

MUST NOT: add OpenSearch because "search exists." MUST NOT: the engine as the source of truth for an order (13 same rule as Redis).

---

## Where it sits

```
modules/<capability>/      # "user may search invoices"; query DTO; authz
infra/search/              # index / delete / query bytes (engine 2 only)
workers/jobs/index_<noun>.py
```

The module chooses the document: ids, denormalised fields the user may see. Infra applies `PUT`/`DELETE`. MUST NOT: `infra/search` invent `orders_v2` as a business event (08).

Writes: after `commit()` of the source row, enqueue index/delete (11), or [Extra 06](06-outbox.md) if a missed index is a defect. MUST NOT: update OpenSearch inside the request session as the only path (16: do not hold Postgres while the engine waits — and do not dual-write without an outbox if you cannot rebuild).

Rebuild: a job that pages Postgres (`LIMIT`, 16) and upserts. Re-runnable.

---

## Query path

```
GET /v1/orders/search?q=…     # module router
  → OrderService.search
      → authz (tenant/user) first or as a filter the engine cannot be trusted to own
      → infra/search.query or repo FTS
      → return {items, next_cursor, limit} (12)
```

MUST: every hit still passes the owning **service** (or a filter that includes `user_id` / `tenant_id` [Extra 01](01-multi-tenant.md)). MUST NOT: return a hit the list API would 404. Same 404 for "no access" vs empty is for **single** GET; a search list omits forbidden ids — MUST NOT: leak a snippet from another tenant.

MUST NOT: resolver → engine from `http/`. MUST NOT: raw engine query string from the client (injection). The service builds the query object.

`limit` capped in `config/` (16). Timeouts on the engine client (16). Down → `ServiceUnavailableError` (05), not an empty list that looks like "no matches."

---

## Documents

Store in the engine: fields needed to rank and display the hit. MUST NOT: password_hash, tokens, other users' PII the hit card does not show.

Map `id` = Postgres UUID. Delete on source delete (or [Extra 10](10-retention.md): follow that Extra's erase vs hide).

---

## Done

- [ ] Postgres FTS first unless an engine is required; folder is `infra/search/`, not the vendor
- [ ] Postgres remains source of truth; index is rebuildable
- [ ] Index/delete via job (or [Extra 06](06-outbox.md)); query through the owning service
- [ ] List envelope (12); no forbidden hits; client cannot send raw engine DSL
