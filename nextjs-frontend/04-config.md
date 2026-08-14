# 04 · Config

WHEN: reading an env var, adding `NEXT_PUBLIC_*`, a URL, or a timeout the UI uses.
LOAD: this file only.
RELATED: 03 (`lib/env.ts`) · 09 (API base URL) · 15 (secrets) — open only if the task is also that topic.
SCOPE: `frontend/lib/env.ts` (and `.env.example`). Not FastAPI `Settings` — that is the other stack.

---

## Two kinds of value

Stop at the first yes.

1. The browser must not see it (API secret, service token, private webhook)?
   → **not in this app.** It belongs on FastAPI or the host. MUST NOT: `NEXT_PUBLIC_`.
2. The **server** of this Next process needs it (FastAPI base URL for RSC fetch, cookie name)?
   → server env. Read only in Server Components, Route Handlers, or `lib/` functions with no `"use client"`.
3. The **browser** must see it (public CDN host, public posthog key that is designed to be public)?
   → `NEXT_PUBLIC_*`. Treat it as public forever.

MUST: parse once in `lib/env.ts` (Zod). Typed exports: `serverEnv`, `publicEnv`. MUST NOT: `process.env.FOO` in a feature file.

MUST NOT: a default API secret. MUST NOT: `NEXT_PUBLIC_API_TOKEN`.
MUST NOT: log env objects (15).

`.env.example` lists every name, empty. `.env.local` is git-ignored.

---

## What lives here

- `API_BASE_URL` — server-side FastAPI origin (`http://api:8000` in compose, public URL if RSC fetches from the edge). From `config`/host.
- `NEXT_PUBLIC_APP_URL` — canonical origin for metadata (06).
- Timeouts and list `limit` caps the UI will send as query — numbers, not policy ("max 50 line items" is the API).

Cookie **names** may be here. Cookie **secrets** must not.

---

## Done

- [ ] No `process.env` outside `lib/env.ts`
- [ ] No secret on `NEXT_PUBLIC_`
- [ ] `.env.example` has names only
