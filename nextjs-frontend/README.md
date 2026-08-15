# Next.js frontend

WHEN: any TypeScript/React/Next.js UI in this stack (`frontend/` app tree).
LOAD: [AGENTS.md](../AGENTS.md) first. Then [agents/02-turn.md](../agents/02-turn.md). Then this map. Then **one** numbered file below that matches the task (+ that file's `LOAD:` line only). On session start also LOAD [agents/01-boundary.md](../agents/01-boundary.md). On error LOAD [agents/04-errors.md](../agents/04-errors.md). Visual work: [01-design.md](01-design.md) **before** other frontend files.

MUST NOT: load all sixteen files for one change.
MUST NOT: load Extra unless a line under Extra matches what this product **already** has.
MUST NOT: copy [python-fastapi-backend](../python-fastapi-backend/README.md) folders into `frontend/`.
MUST NOT: load anything under `for-humans/` unless `@`-referenced.

TARGETS: **Next.js App Router** (RSC default, Server Actions, `proxy.ts` — the file Next 16 renamed from `middleware.ts`), React 19, TypeScript `strict`. WHEN: the product is on an older major, or Next ships a new one — `05`, `06`, `07`, `08`, `12` are the rules that encode framework behaviour; re-read those and adapt in git ([AGENTS.md](../AGENTS.md), *Not carved in stone*). MUST NOT: apply a rule this project's installed major does not have (Pages Router, or a Next without Server Actions, is a different playbook).

Rule strength (full definition: [AGENTS.md](../AGENTS.md)): `[critical]` = secret, another user's data, or an authz gate — state the exposure before writing the code. Unmarked `MUST` = this playbook's shape; follow by default, yield to the user and to what the product already ships. `SHOULD` = a default, deviate with a reason.

---

## Agent: which numbered file WHEN

WHEN: any screen, colour, spacing, empty/loading/error state, or generated UI.
LOAD: [01-design.md](01-design.md) **first**.

WHEN: split file, types, naming, no `utils/`.
LOAD: [02-coding-principles.md](02-coding-principles.md).

WHEN: new file and unsure `app/` vs `features/` vs `ui/` vs `lib/`.
LOAD: [03-file-structure.md](03-file-structure.md).

WHEN: adding a package, `package.json`, lockfile, Node version, an audit finding, or a framework upgrade.
LOAD: [03-file-structure.md](03-file-structure.md) Dependencies + [agents/03-anti-patterns.md](../agents/03-anti-patterns.md) Dependencies **before** the manifest edit.

WHEN: env var, `NEXT_PUBLIC_*`, server vs browser env.
LOAD: [04-config.md](04-config.md).

WHEN: `"use client"`, hooks, event handler, server vs client.
LOAD: [05-server-client.md](05-server-client.md).

WHEN: URL, layout loading/error, metadata, `noindex`.
LOAD: [06-routing.md](06-routing.md).

WHEN: server fetch, RSC data, list/detail read — not client GET in `useEffect`.
LOAD: [07-data.md](07-data.md).

WHEN: form, mutation, Server Action, save/cancel/delete.
LOAD: [08-forms.md](08-forms.md).

WHEN: API client, `/v1`, cookies, `error_code`, base URL.
LOAD: [09-api-client.md](09-api-client.md).

WHEN: new feature under `features/<noun>/`.
LOAD: [10-features.md](10-features.md).

WHEN: Button, field, modal, skeleton, token, primitive.
LOAD: [11-ui.md](11-ui.md) (+ [01-design.md](01-design.md)).

WHEN: sign-in, session cookie, protected route.
LOAD: [12-auth.md](12-auth.md).

WHEN: URL params, global store, "need state".
LOAD: [13-state.md](13-state.md).

WHEN: tests, four states, mirror path.
LOAD: [14-testing.md](14-testing.md).

WHEN: PR pass, XSS, CSRF, secrets, a11y before merge.
LOAD: [15-security.md](15-security.md).

WHEN: slow page, bundle, images, RSC waterfall.
LOAD: [16-performance.md](16-performance.md).

WHEN: user pasted build/runtime/hydration/API error.
LOAD: [agents/04-errors.md](../agents/04-errors.md) first; then the numbered file named in the matched block.

MUST NOT: open `RELATED:` unless the task is also that topic.

## Backbone

```
app/         URL, layout, loading/error. no product rule.
features/    one product noun per package.
ui/          primitives. no product sentence.
lib/         env, fetch wrapper. not utils/.
```

MUST [critical]: FastAPI owns writes and authz. MUST NOT [critical]: Next talking to Postgres. MUST NOT [critical]: JWT in `localStorage`.

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
