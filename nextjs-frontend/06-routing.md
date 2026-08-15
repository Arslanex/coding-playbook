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
- `error.tsx` — client boundary; human `message`, not a stack (09, 01). It also **reports** — below.
- `global-error.tsx` — the root boundary, for a failure in `layout.tsx` itself. Without it, that failure is a blank page.
- `not-found.tsx` — same visual language as empty (01).

Route groups `(app)` / `(marketing)` for layouts. MUST NOT: `(app)` as a reason to dump components in `app/`.

MUST: `page.tsx` is a Server Component unless 05's first yes fired for the **whole** page (it almost never should).

---

## Reporting (the error has to reach you, not just the user)

An `error.tsx` that renders a nice message and reports nothing means the user knows and you do not. In production nobody is reading the browser console.

MUST: one reporting sink for the app, initialised once, configured from `lib/env.ts` (04). The project picks the tool; this file only requires that one exists and that a failure arrives somewhere a human looks.

MUST: report from all three sources — they do not overlap.

1. **Render errors** — `error.tsx` and `global-error.tsx` report before rendering the fallback. React does not do it for you.
2. **API errors** — from the wrapper, not from each caller (09). Which ones are worth reporting is below.
3. **Unhandled** — a rejected promise or a thrown value nothing caught. Register the global handler once in the root layout's client entry, not per page.

MUST: attach `X-Request-ID` to the report (09). It is the only thing that joins a browser incident to a backend log line (backend 04).

MUST NOT: report every `ApiError`. A 401 that sends the user to sign-in, a 404 for a record they typed wrong, a 422 the form displays on the field — these are the product working. Reporting them buries the one report that mattered under a thousand that did not.
MUST: report 5xx, a network failure, a response that failed Zod parsing (the contract broke), and anything the UI could not turn into a state a user can act on.

MUST NOT: put PII in a report — no email, no token, no full request body (15). The `error_code`, the route, and the request id are enough to find it.
MUST NOT: a reporting call in a Server Component that silently swallows the render. Report, then re-throw or return the error UI.
MUST NOT: `console.error` as the reporting strategy. It is a debugging tool, not a sink.

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
- [ ] `error.tsx` / `global-error.tsx` exist and report, not just render
- [ ] Reports carry the request id; expected 4xx are not reported as incidents
- [ ] Public URL has title/description; app URL is `noindex`
- [ ] No `/v1` in the Next path
