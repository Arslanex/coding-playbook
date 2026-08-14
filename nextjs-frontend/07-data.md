# 07 · Data (reads)

WHEN: loading a resource for the first paint, a list, a detail, or cache/revalidate.
LOAD: this file. Server vs client: 05. API shape: 09. Lists: backend 12 cursor.
RELATED: 08 (writes) · 13 (client cache) — open only if the task is also that topic.
SCOPE: how this app **reads** FastAPI. MUST NOT: Prisma / SQL / Redis from Next.

The first paint is a Server Component `fetch` (or the wrapper in `lib/api.ts`). The browser is not the source of truth.

---

## First load

MUST: the page or a server feature module `await`s the API. Cookies go on that request (12).

MUST NOT: `useEffect(() => { fetch(...) }, [])` for the first load.
MUST NOT: React Query / SWR as the default for the first load. Add them only when the same view must refetch in the browser without a navigation (13). Until then: server fetch + `router.refresh()` after a write (08).

Parse with the feature Zod schema (09, 10). `unknown` in, typed out. MUST NOT: `data as Order`.

---

## Cache

`fetch` to FastAPI: set an explicit cache policy. Default for user-specific data: `cache: "no-store"` (or `cookies()` already opts you out). Public, rarely changing: `next: { revalidate: n }` from `lib/env.ts` (04).

MUST NOT: cache a user's orders as a static page.
MUST NOT: `force-cache` on an authenticated request.

---

## Lists

The API returns `{ items, next_cursor, limit }`. The UI pages with the cursor, not `page=1` unless the API already does (it should not — backend 12).

Four states (01): loading skeleton, empty, error (`error_code` + `message`), list.

---

## Done

- [ ] First paint is a server fetch, parsed with Zod
- [ ] No `useEffect` GET
- [ ] User data is not statically cached
- [ ] Cursor list, not a fake infinite dump
