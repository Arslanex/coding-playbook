# Next.js frontend

WHEN: any TypeScript/React/Next.js UI in this stack (`frontend/` app tree).
LOAD: [the playbook root](../README.md) first (not carved in stone — adapt to this project and write it back here). Then this map, then the numbered file that matches the task, plus whatever that file's own `LOAD:` line names. Visual work: [01-design.md](01-design.md) **before** other files.

MUST NOT: load all sixteen files for one change.
MUST NOT: load Extra unless a line under Extra matches what this product **already** has.
MUST NOT: copy [python-fastapi-backend](../python-fastapi-backend/README.md) folders into `frontend/` (`http/` · `modules/` · `infra/` are the API, not the UI).

## Backbone

```
app/         URL, layout, loading/error. no product rule.
features/    one product noun per package.
ui/          primitives. no product sentence.
lib/         env, fetch wrapper. not utils/.
```

MUST: FastAPI owns writes and authz. MUST NOT: Next talking to Postgres. MUST NOT: JWT in `localStorage`.

## How to load

Each numbered file carries two lines:

- `LOAD:` — what you must have open to do this task correctly.
- `RELATED:` — siblings to open **only if the task is also that topic**.

## Files

1. [01-design.md](01-design.md) — AI-slop, ratios, colour, four states, UX
2. [02-coding-principles.md](02-coding-principles.md) — one job, types, no `utils/`
3. [03-file-structure.md](03-file-structure.md) — `app/` vs `features/` vs `ui/` vs `lib/`
4. [04-config.md](04-config.md) — `lib/env.ts`, `NEXT_PUBLIC_` vs server
5. [05-server-client.md](05-server-client.md) — `"use client"` is a leaf
6. [06-routing.md](06-routing.md) — URLs, loading/error, metadata / `noindex`
7. [07-data.md](07-data.md) — server fetch first; no `useEffect` GET
8. [08-forms.md](08-forms.md) — writes go to FastAPI; thin Server Actions
9. [09-api-client.md](09-api-client.md) — `/v1`, cookies, `error_code`
10. [10-features.md](10-features.md) — one noun per package
11. [11-ui.md](11-ui.md) — tokens and primitives
12. [12-auth.md](12-auth.md) — HttpOnly cookie; no JWT-as-UI
13. [13-state.md](13-state.md) — URL first; no god store
14. [14-testing.md](14-testing.md) — mirror path; four states
15. [15-security.md](15-security.md) — XSS, CSRF, secrets, a11y pass
16. [16-performance.md](16-performance.md) — RSC, images, no client waterfall

Order: look → write → where → env → server/client → URL → read → write → API → feature → primitive → session → state → test / security / speed.

## Extra

If the product **already** has that shape, LOAD the matching file. 01–16 still apply.

MUST NOT: load Extra because it might be needed later. MUST NOT: scaffold `frontend/extra/` or `app/i18n/` as a backbone.

1. Tenant in the URL — [extra/01-multi-tenant.md](extra/01-multi-tenant.md)
2. Two or more Next apps — [extra/02-apps.md](extra/02-apps.md)
3. More than one language — [extra/03-i18n.md](extra/03-i18n.md)
4. IdP sign-in in the browser — [extra/04-sso.md](extra/04-sso.md)
5. Live SSE / WebSocket — [extra/05-realtime.md](extra/05-realtime.md)
6. Search screen — [extra/06-search.md](extra/06-search.md)
7. File upload UX — [extra/07-uploads.md](extra/07-uploads.md)
8. Agent run transcript — [extra/08-agents.md](extra/08-agents.md)

Same list: [extra/README.md](extra/README.md).
