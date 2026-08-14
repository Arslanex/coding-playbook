# 03 · File structure

WHEN: creating a file, folder, or deciding where UI code belongs.
LOAD: this file. How to write: [02-coding-principles.md](02-coding-principles.md). Visual: [01-design.md](01-design.md) if the screen is new.
RELATED: 05 (server vs client) · 06 (`app/`) · 10 (`features/`) · 11 (`ui/`) — open only if the task is also that topic.
SCOPE: the application `frontend/` tree. This playbook folder is `nextjs-frontend/`; do not name the app after it.

MUST: first matching yes in Decide. MUST NOT: add a top-level folder not in the tree.
MUST NOT: `utils/` · `helpers/` · `common/` · `shared/` · `components/` as a junk drawer.
MUST NOT: copy Extra into `frontend/` because Extra exists.

---

## Tree

```
frontend/
├── package.json
├── next.config.ts
├── tsconfig.json
├── .env.example
├── app/
│   ├── layout.tsx
│   ├── not-found.tsx
│   ├── error.tsx
│   ├── globals.css          # tokens + reset only
│   └── (routes)/            # groups. pages import features
├── features/
│   └── <noun>/
├── ui/                      # primitives. no product noun
├── lib/
│   ├── env.ts
│   └── api.ts               # fetch wrapper (09)
└── tests/                   # mirrors features/, ui/, lib/ — not app/ as a dump
```

Leaves shown are fixed. Other names: the numbered file for that folder.

---

## Decide

Unknown unit: ask in order. Stop at the first yes.
Product noun = `orders`, `auth`, `billing`. Not `component`, `hook`, `util`.

1. A URL segment, layout, `loading.tsx`, `error.tsx`, `not-found.tsx`, or metadata for a route?
   → `app/`. Details: [06-routing.md](06-routing.md). The page file composes; it does not fetch-and-render 200 lines of JSX.
2. A primitive with no product sentence (Button, TextField, Skeleton, Empty)?
   → `ui/`. Details: [11-ui.md](11-ui.md).
3. Env, fetch wrapper, cookie helpers, `cn` if one exists?
   → `lib/`. Details: 04, 09, 12. MUST NOT: `lib/orders.ts`.
4. Product UI, Zod for that resource, the function that calls `/v1/orders`?
   → `features/<noun>/`. Details: [10-features.md](10-features.md).
5. Test?
   → `tests/` mirroring `features/` / `ui/` / `lib/` (14).

No yes: not a new top-level folder. Put a named file in the existing `features/<noun>/` (02).

MUST NOT: `app/orders/components/` as a second feature tree. The route file imports `features/orders`.
MUST NOT: `src/` **and** `app/` as two cities — Next `app/` is the route city; features sit **beside** `app/` at `frontend/` root (or under `frontend/src/` only if the whole tree is in `src/`; pick one in `tsconfig` and stay). Default: `frontend/app` + `frontend/features` (no extra `src/`).

---

## Root files

`package.json` — dependencies **and** scripts. Pin Next, React, TypeScript. MUST NOT: a second lockfile story.

`next.config.ts` — images, redirects, headers (15). MUST NOT: business rules.

`.env.example` — names, empty values (04). `.env*` with secrets is git-ignored.

---

## Done

- [ ] Decide first-yes matches the path
- [ ] No `utils/` / `components/` dump / `features` inside `app/`
- [ ] Page in `app/` is thin; product JSX in `features/`
- [ ] Primitive has no product noun
