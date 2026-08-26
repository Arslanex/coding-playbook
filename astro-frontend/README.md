# Astro frontend

WHEN: any edit under the product `frontend/` tree in this repo (`.astro`, its browser scripts, its CSS).
LOAD: [AGENTS.md](../AGENTS.md) first. Then [agents/02-turn.md](../agents/02-turn.md). Then this map, then the **one** file below that matches the task.
SCOPE: Astro with a server adapter (SSR), rendering pages per request from the FastAPI content API. Written because [nextjs-frontend](../nextjs-frontend/README.md) encodes App Router behaviour — RSC, Server Actions, `next/image`, `proxy.ts` — that Astro does not have, and [AGENTS.md](../AGENTS.md) *Stacks* says to add a stack folder rather than bend another stack's rules onto the work.

TARGETS: Astro 5 with `output: "server"` and `@astrojs/node`, TypeScript `strict`, no UI framework — plain `.astro` templates plus small vanilla-JS behaviours. WHEN: Astro ships a major — re-read [01-ssr-data.md](01-ssr-data.md), which is the file that encodes framework behaviour.

MUST NOT: load all of `nextjs-frontend/` "for reference".
MUST NOT: add React, Vue, or Svelte to render a page that renders fine as HTML.

---

## What this stack borrows unchanged

These `nextjs-frontend` files are framework-independent. LOAD them from there; do not copy them here.

| File | Applies |
|---|---|
| [01-design.md](../nextjs-frontend/01-design.md) | in full — load **first** for any visual work |
| [02-coding-principles.md](../nextjs-frontend/02-coding-principles.md) | in full (`utils/` ban, one job per file, types at boundaries) |
| [11-ui.md](../nextjs-frontend/11-ui.md) | tokens and primitives; here they are CSS custom properties and `.astro` partials, not React components |
| [14-testing.md](../nextjs-frontend/14-testing.md) | mirror path, four states |
| [15-security.md](../nextjs-frontend/15-security.md) | in full. `set:html` is this stack's `dangerouslySetInnerHTML` — same rule |
| [16-performance.md](../nextjs-frontend/16-performance.md) | the Decide list; "client bundle" means the `<script>` a page ships |

## What this stack replaces

| Instead of | Read |
|---|---|
| `03-file-structure` (`app/` + `features/`) | *Backbone* and *Decide* below |
| `04-config` (`NEXT_PUBLIC_*`) | *Config* below |
| `05-server-client`, `07-data`, `09-api-client`, `13-state` | [01-ssr-data.md](01-ssr-data.md) |
| `06-routing` (App Router) | *Routing* below |
| `08-forms` (Server Actions) | [01-ssr-data.md](01-ssr-data.md) *Writes* |
| `10-features` (`features/<noun>/`) | *Backbone* below — a marketing site's unit is a page section, not a product noun |
| `12-auth` | nothing: this stack has no visitor session. WHEN one appears, write the file before the code |

---

## Backbone

```
frontend/
├── package.json
├── astro.config.mjs
├── tsconfig.json
├── .env.example
├── public/              # served as-is: fonts, favicons, design images, robots.txt
└── src/
    ├── pages/           # one file per URL. Fetches, then composes. No CSS rule, no product rule
    ├── layouts/         # the page shell: head, nav, footer, script and style includes
    ├── components/      # one section of one page (Hero.astro, Partners.astro)
    ├── lib/             # env, the API client, content types. Not `utils/`
    ├── styles/          # global CSS. Design tokens in one file
    ├── scripts/         # browser behaviour, one file per page or per widget
    └── assets/          # images Astro optimises at build time (design assets, not CMS media)
```

MUST NOT: `utils/` · `helpers/` · `common/` · a second `components/` under `pages/`.
MUST NOT: copy backend folder names (`http/`, `modules/`, `infra/`) into `frontend/`.
MUST NOT [critical]: the frontend talking to PostgreSQL or to Airtable. It reads the API (`/v1/...`) and nothing else.

## Decide

Stop at the first yes.

1. A URL, or the `<head>`/metadata of one?
   → `src/pages/` (a URL) or `src/layouts/` (the shell). A page **composes**; it does not hold 200 lines of markup.
   A header that must be on **every** response (cache policy, security headers) is `src/middleware.ts` — one file, no product noun.
2. One visible section of a page, taking its content as props?
   → `src/components/`. Name it after the section (`ImpactMosaic.astro`), never `Section1.astro`.
3. Reads `import.meta.env`, calls the API, or declares a content type?
   → `src/lib/`. One file per job: `env.ts`, `api.ts`, `content.ts`.
4. A CSS rule?
   → `src/styles/`. MUST: a token (colour, spacing, radius) is defined once, in the tokens file.
5. Behaviour that must run in the browser (rotator, counter, flip card, menu)?
   → `src/scripts/`, loaded by the page that needs it. MUST NOT: a framework island for what 30 lines of DOM code does.
6. An image the design ships?
   → `src/assets/` when Astro should optimise it; `public/` when it must keep its exact path.
   MUST NOT: CMS media in either — that arrives as a URL from the API (`/media/...`).

No yes: a named file inside the component or page that owns it.

## Routing

One file in `src/pages/` per URL. The URL is the filename; there is no route group and no `layout.tsx` chain — a page imports its layout.

MUST: every page sets `<title>` and a description through the layout, from content the API returned.
MUST: a 404 page exists (`src/pages/404.astro`) and a 500 page when the adapter serves one.
MUST: `sitemap.xml` and `robots.txt` list the URLs that exist. In SSR the sitemap integration only sees prerendered routes, so a handful of pages is a hand-written endpoint, not a plugin.
MUST NOT: a URL that exists only in the nav.

## Config

MUST: one file, `src/lib/env.ts`, reads `import.meta.env` and exports a typed, validated object. Everything else imports from there.
MUST NOT: `import.meta.env` in a page, a component, or a browser script.
MUST [critical]: a value the browser must not see never reaches a `PUBLIC_*` name and never reaches a `<script>`. In this stack the API base URL is read **on the server**; the browser talks to the same origin through the reverse proxy.
MUST: `.env.example` lists every name with an empty value; `.env` is git-ignored.

Unlike `NEXT_PUBLIC_*`, an SSR page reads server env **at request time** — so a config change is a restart, not a rebuild. MUST: keep it that way; a value baked into the client bundle is a rebuild, and this stack exists to avoid rebuilds.

---

## Done (before writing frontend code)

- [ ] The task's file matches Decide's first yes
- [ ] Visual work read [01-design.md](../nextjs-frontend/01-design.md) first
- [ ] Data and caching follow [01-ssr-data.md](01-ssr-data.md)
- [ ] No framework island added for DOM work
- [ ] No `import.meta.env` outside `src/lib/env.ts`
- [ ] Nothing in the browser bundle that the server should have kept
