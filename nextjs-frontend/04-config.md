# 04 · Config

WHEN: reading an env var, adding `NEXT_PUBLIC_*`, a URL, or a timeout the UI uses.
LOAD: this file only.
RELATED: 03 (`lib/env.ts`) · 09 (API base URL) · 15 (secrets) — open only if the task is also that topic.
SCOPE: `frontend/lib/env.ts` (and `.env.example`). Not FastAPI `Settings` — that is the other stack.

---

## Two kinds of value

Stop at the first yes.

1. The browser must not see it (API secret, service token, private webhook)?
   → **not in this app.** It belongs on FastAPI or the host. MUST NOT [critical]: `NEXT_PUBLIC_`.
2. The **server** of this Next process needs it (FastAPI base URL for RSC fetch, cookie name)?
   → server env. Read only in Server Components, Route Handlers, or `lib/` functions with no `"use client"`.
3. The **browser** must see it (public CDN host, public posthog key that is designed to be public)?
   → `NEXT_PUBLIC_*`. Treat it as public forever.

MUST: parse once in `lib/env.ts` (Zod). Typed exports: `serverEnv`, `publicEnv`. MUST NOT: `process.env.FOO` in a feature file.

MUST NOT [critical]: a default API secret. MUST NOT [critical]: `NEXT_PUBLIC_API_TOKEN` — a public env var is in the JS bundle forever, and rotating it means rebuilding every deploy that shipped it.
MUST NOT [critical]: log env objects (15).

`.env.example` lists every name, empty. `.env.local` is git-ignored.

---

## When the value is read (build vs runtime)

`NEXT_PUBLIC_*` is **inlined into the bundle at `next build`**, not read at boot. One image promoted staging → prod carries staging's public values forever; changing the host env var changes nothing until the app is rebuilt.

MUST: `NEXT_PUBLIC_*` only for values that are the **same for every deploy of that build**, or accept one build per environment.
MUST NOT: expect a `NEXT_PUBLIC_*` change on the host to take effect without a rebuild — that is a rebuild, not a restart.
MUST NOT: bake a per-environment public value into a shared image and call it configuration.

Server env (no `NEXT_PUBLIC_`) **is** read at runtime — `API_BASE_URL` and anything else that differs per environment belongs there, reached from Server Components / Route Handlers (05). If the browser genuinely needs a per-environment value, pass it **down as a prop** from a Server Component, or serve it from a small Route Handler. MUST NOT: a second public env var to work around this.

Build args: a Docker build that needs `NEXT_PUBLIC_*` takes them as `ARG`/`ENV` **before** `next build`. `.env.example` says which ones are build-time.

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
- [ ] Per-environment values are server env (runtime), not `NEXT_PUBLIC_` (build-time)
- [ ] `.env.example` has names only, and marks the build-time ones
