# 05 · Server vs client

WHEN: adding `"use client"`, a hook, an event handler, or asking "does this file run on the server?".
LOAD: this file. Placement: 03. Fetch: 07. Forms: 08.
RELATED: 06 (layouts) · 12 (auth on the server) — open only if the task is also that topic.
SCOPE: the Next.js App Router split. Default is a Server Component.

A Client Component is a privilege. It is not the default for "React."

---

## Decide

Stop at the first yes.

1. The file handles a browser event (`onClick`, `onChange`), `useState`, `useEffect`, or a browser-only API (`window`, `localStorage`)?
   → `"use client"` at the **top of the leaf that needs it**, not on the page.
2. The file reads cookies/headers and fetches FastAPI for the first paint?
   → Server Component. No `"use client"`.
3. The file is a `ui/` primitive that must take `onClick`?
   → `"use client"` on that primitive, or pass `children` from a tiny client wrapper. MUST NOT: mark the whole `features/orders` tree client because one button exists.

MUST NOT: `"use client"` on `app/**/page.tsx` "so hooks work." Split: server `page.tsx` fetches and renders; a client child owns the interactive island.

MUST NOT [critical]: import a server-only module (`lib/env.ts` server secrets, `fs`) into a client file — the value ends up in the browser bundle. Mark those modules `import "server-only"` so the build fails instead of shipping the secret.

---

## Boundary

```
page.tsx (server)
  → features/orders/OrderList (server, awaits fetch)
       → ui/Button (client, onClick)
       → features/orders/CancelButton (client, calls POST helper)
```

Data flows **down** as props (serialisable). MUST NOT: pass a function from server to client except as a Server Action that is only a thin POST (08).

---

## Done

- [ ] `"use client"` only on the leaf that needs events or browser APIs
- [ ] First paint data still fetched on the server (07)
- [ ] No server secrets imported into a client module
