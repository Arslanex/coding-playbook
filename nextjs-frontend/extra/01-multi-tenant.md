# Extra 01 · Multi-tenant (UI)

WHEN: the signed-in URL includes a tenant slug (or host) and two customers must not see each other's chrome.
LOAD: this file **and** 06, 09, 12. Backend: [python-fastapi-backend/extra/01-multi-tenant.md](../../python-fastapi-backend/extra/01-multi-tenant.md). Not instead of 01–16.
SCOPE: how the slug gets into the path and the API. MUST NOT: `features/tenant/` as a second backbone. MUST NOT: tenant from JWT in the browser (12).

---

## URL

MUST: slug from the **path** (`/t/[slug]/orders`) or from a host the product already owns. `app/t/[slug]/…` layouts wrap the app group.

MUST NOT: `?tenant_id=` as the authority. MUST NOT: a client store as the only tenant (13).

`features/tenants` — switcher, list memberships. Other features receive `slug` as a prop/param and send it to FastAPI the way the backend Extra says (header or path the API documents). The **API** still filters; hiding a nav item is not isolation (15).

404 for another tenant's id: same empty/not-found as missing (01, backend 15).

---

## Done

- [ ] Slug from URL/host; not from a decoded JWT
- [ ] Features pass slug into `api.ts`; they do not invent isolation
- [ ] Switcher is `features/tenants`, not `ui/TenantBar` as a primitive with baked orgs
