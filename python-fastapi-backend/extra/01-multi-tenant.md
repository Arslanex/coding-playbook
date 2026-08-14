# Extra 01 · Multi-tenant

WHEN: two or more customers must not see each other's rows, and that is a **shipped** requirement.
LOAD: this file **and** [06](../06-database.md), [09](../09-modules.md), [10](../10-http.md), [13](../13-identity-security.md), [15](../15-security.md). Not instead of them.
SCOPE: how tenant isolation is **added** to the existing tree. This playbook's default has no tenant (04, 06, 13).

Tenant is a product noun (the customer org). It is not a mixin you sprinkle on day one of Extra, and it is not `shared/`.

---

## When to add

Add only if deleting tenant isolation would leak data between customers.

MUST NOT: add `tenant_id` "because SaaS." One customer = no tenant column.
MUST NOT: copy this product's `/v1/t/{slug}/` tree into a single-tenant app.

---

## How it attaches to the backbone

Do not add a top-level `tenant/` city. Extend what exists.

`modules/tenants/` — org as a capability: create tenant, slug, membership. Day-one files (09): router, schemas, service.

`http/deps` — after `get_current_user`, `get_tenant`: resolve **from the URL** (`/v1/t/{slug}/…`), then check membership. MUST NOT: take `tenant_id` from the JWT as the only source (stale, forgeable). MUST NOT: take `tenant_id` from the JSON body as the authority.

`infra/db` — `tenant_id` UUID on tables that belong to an org. Repositories: **every** SELECT/UPDATE/DELETE includes `tenant_id` from the service. Service passes it; repo does not guess. MUST NOT: a query that lists by id globally then "filters in Python."

Same 404 for missing, not-owned, and **other tenant's** id (15). A 403 would confirm the row exists.

Cache / rate-limit / lock keys include `tenant_id` or `slug` (13, 10). Queue payload includes `tenant_id`; the **service** re-checks membership/status when the job runs (11) — the row may have moved or been disabled while queued.

Logs: bind `tenant_id` like `user_id` (04) — null until resolved. MUST NOT: require it on a playbook that still has no tenants.

---

## Models

When this Extra is on:

- `Tenant` — `id`, `slug` (unique), timestamps. Write owner: `TenantService`.
- `TenantMembership` — `tenant_id`, `user_id`, role-in-org if needed. User accounts stay global (`User` in 13); org access is membership.
- Org-owned tables (`Order`, …) — `tenant_id` FK, indexed. MUST NOT: nullable "sometimes platform-wide" without an explicit platform owner.

Mixin: if **five or more** tables share the exact `tenant_id` column, `TenantColumns` in `models/base.py` (06). Still not `TenantMixin` as a behavior bag — columns only. MUST NOT: mixin methods that encode "current tenant."

Default Extra shape: **one Postgres**, `tenant_id` on rows. A database **per** tenant is a later split ([Extra 02](02-microservices.md)) — only if isolation/compliance requires it. Then `session.py` still opens one session per request; the resolver picks the DSN. MUST NOT: two sessions for one use case (06).

---

## What not to build

MUST NOT: `shared/tenant.py` toolbox. MUST NOT: RLS as the **only** gate — application filter stays primary (15). MUST NOT: `modules/common/` for "tenant helpers." Other modules call `TenantService` or receive `tenant_id` from deps into **their** service.

Tests (14): every org query test has a second tenant that must not appear. Factory always sets `tenant_id`.
