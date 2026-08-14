# 06 · Routing and metadata

WHEN: a URL, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, redirect, or `<title>` / Open Graph.
LOAD: this file. Visual: [01-design.md](01-design.md) if the frame is new. Auth gates: 12.
RELATED: 03 (where `app/` sits) · 07 (what the page fetches) · 15 (`noindex`, headers) — open only if the task is also that topic.
SCOPE: `frontend/app/`. Product JSX still lives in `features/` (10).

`app/` is the URL shell. It does not know how to cancel an order.

---

## Files in `app/`

- `layout.tsx` — html/body, fonts, providers that must wrap everything (keep providers tiny — 13).
- `page.tsx` — compose a feature. MUST NOT: 200 lines of markup.
- `loading.tsx` — skeleton that matches the layout (01).
- `error.tsx` — client boundary; human `message`, not a stack (09, 01).
- `not-found.tsx` — same visual language as empty (01).

Route groups `(app)` / `(marketing)` for layouts. MUST NOT: `(app)` as a reason to dump components in `app/`.

MUST: `page.tsx` is a Server Component unless 05's first yes fired for the **whole** page (it almost never should).

---

## URLs

MUST: human paths (`/orders`, `/orders/[id]`). MUST NOT: `/v1` in the Next path — that is the API (09).
MUST NOT: a verb as a folder (`app/cancel/`). Cancel is an action on the order feature.

Colocation: `app/orders/page.tsx` imports `features/orders`. MUST NOT: `app/orders/OrderTable.tsx`.

---

## Metadata (SEO when the URL is public)

Public documents (marketing, share link, hub): MUST set `title`, `description`, canonical via `metadata` / `generateMetadata`. `NEXT_PUBLIC_APP_URL` + path (04).

Auth and in-app screens: MUST: `robots: { index: false, follow: false }` (or equivalent). MUST NOT: index a dashboard.

MUST NOT: fake OG images with lorem (01). MUST NOT: a client-only public page whose HTML is empty — crawlers see nothing (05, 07).

Sitemap / JSON-LD: Extra when the product is a public site. Until then: metadata on the public routes that exist. MUST NOT: a `sitemap.ts` of private URLs.

---

## Done

- [ ] `app/` is thin; feature owns the screen
- [ ] loading / error / not-found exist for that segment when the layout is non-trivial
- [ ] Public URL has title/description; app URL is `noindex`
- [ ] No `/v1` in the Next path
