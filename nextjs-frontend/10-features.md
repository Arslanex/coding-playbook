# 10 · Features

WHEN: a product noun on screen, a new `features/<noun>/`, or code two screens both need.
LOAD: this file. Placement: 03. How to write: 02. API functions: 09.
RELATED: 06 (thin `page.tsx`) · 11 (primitives only) — open only if the task is also that topic.
SCOPE: `frontend/features/<noun>/`. Same idea as backend `modules/` — not the same folders.

A feature is one product noun and everything that **means** on the client.

---

## Package name

English, kebab or snake matching the URL: `orders`, `auth`. MUST NOT: `components`, `views`, `containers`, `orders-feature`.

New folder only if deleting it removes a user-visible ability.

---

## Files (day one)

```
features/orders/
  api.ts          # typed calls (09)
  schemas.ts      # Zod
  OrderList.tsx   # server-friendly panel
  CancelButton.tsx  # client leaf if needed (05)
```

Add a file when 02 split fires. MUST NOT: `index.ts` that re-exports the world. MUST NOT: `hooks.ts` bag. MUST NOT: `types.ts` plus `schemas.ts` that duplicate — Zod infers the type.

`page.tsx` imports these. MUST NOT: the feature import `app/`.

---

## Two features

Same Decide as backend 09: one owner. The other imports the **panel** or calls the **api** of the owner. MUST NOT: `features/shared/`. MUST NOT: copy `OrderStatusBadge` into `billing/`.

---

## Done

- [ ] Folder is the noun; `page.tsx` only composes
- [ ] `api.ts` + Zod in the feature; no `app/` import
- [ ] No `features/shared/`
