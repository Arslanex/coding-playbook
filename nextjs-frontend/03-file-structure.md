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

Node version: pinned in one place the whole team and CI read (`engines` in `package.json`, plus `.nvmrc` or the equivalent your host uses). MUST NOT: "works on my Node."

---

## Dependencies

Four separate questions — which package, which version, how it is pinned, how it stays current. All four are this file's job; none of them is "later."

**Adding one.** MUST: the package exists, is the one you meant, and is maintained — check the registry before writing the name, not after the import fails. MUST NOT: write a package name from memory into `package.json`. An invented or misremembered name is a name an attacker may have registered ([agents/03-anti-patterns.md](../agents/03-anti-patterns.md)).

MUST: prefer the platform. A date format, a `cn`, a debounce, a UUID is not a dependency (02, 16). Each added package is bundle bytes (16), a supply-chain surface (15), and a thing to keep current forever.

**Choosing the version.** MUST NOT: write a version number from memory. The version you remember is the version that was current while you were trained — by definition old, and old is exactly where the published advisories are. The package name being real is not enough; every rule above passes on a correct name with a stale version.

HOW: let the tool resolve it — `npm install <pkg>` / `pnpm add <pkg>` writes the current version and updates the lockfile in one step. If a manifest must be edited by hand, read the current version from the registry first.
MUST NOT: copy a version from a tutorial, a blog post, an answer, or another project's `package.json`. All four are snapshots of some past day.
MUST NOT: guess a range (`^14`, `~5.2`) around a remembered number. A wrong floor silently installs a vulnerable build.
MUST: this applies hardest to the frameworks you are most confident about — `next`, `react`, `typescript`. Confidence is not currency.

**Pinning.** MUST: one lockfile, committed. Direct dependencies pinned or on a narrow range — the lockfile is the reproducible build, and it is only reproducible if CI installs **from** it (`npm ci` / `pnpm install --frozen-lockfile`, not a bare `install`). MUST NOT: a lockfile the agent regenerates to make an error go away.

**Keeping current.** An outdated dependency with a known advisory is a security finding, not backlog.

MUST: a dependency audit step runs in CI on every PR. MUST: a high-severity advisory on a package that actually ships blocks merge — fix, upgrade, or write the accepted risk and its expiry somewhere in the repo.
MUST: framework majors (Next, React) are a planned upgrade with an owner, not a thing that happens when something breaks. When Next's major moves, re-read this stack's `05`, `06`, `07`, `12` — they encode App Router behaviour that majors change.
MUST NOT: pin a version **forever** to avoid a migration and call it stability. Pinning is for reproducibility, not for avoiding upgrades.

Tooling is the project's choice (the platform's built-in audit command, a bot that opens upgrade PRs, or a scheduled job) — the rule is that the step exists and that a finding has an owner.

---

## Build and release

WHEN: the user asks for a container image or a deploy pipeline.
MUST NOT: write these unasked ([agents/01-boundary.md](../agents/01-boundary.md)).

MUST: decide the build target first — the host builds from source (Vercel, Amplify), or you ship an image (`output: "standalone"` in `next.config.ts`, then a slim runtime stage).

MUST: `NEXT_PUBLIC_*` values are supplied as build arguments **before** `next build` — they are baked in, not read at boot (04). One image per environment, or none of them are public values.
MUST NOT: promote one image across environments while expecting a `NEXT_PUBLIC_*` change to take effect. That is a rebuild.

MUST: `.dockerignore` excluding `.env*`, `.git`, `node_modules`, `.next/cache`.
MUST NOT [critical]: a `.env` file or a server secret inside the image. A public bundle is already public; a server secret in a layer is a leak.
MUST: pin the base image by digest and the Node version the same way local and CI pin it (above).

CI, in this order: install from the lockfile (`npm ci`) → type-check → lint → tests → build → dependency audit.
MUST: the build runs in CI, not only on the host. A build that only fails at deploy time fails in front of users.

---

## Done

- [ ] Decide first-yes matches the path
- [ ] No `utils/` / `components/` dump / `features` inside `app/`
- [ ] Page in `app/` is thin; product JSX in `features/`
- [ ] Primitive has no product noun
- [ ] New package: verified in the registry, resolved by the tool — no hand-typed version
- [ ] Not something the platform already does
- [ ] One committed lockfile; CI installs from it; Node version pinned
- [ ] Audit step in CI; no unowned high-severity advisory
