# 01 · SSR, data, and caching

WHEN: a page fetches content, a write leaves the browser, a value is cached, or you are deciding what runs on the server and what runs in the browser.
LOAD: this file. Placement: [README.md](README.md). Visual states: [nextjs-frontend/01-design.md](../nextjs-frontend/01-design.md).
RELATED: [nextjs-frontend/16-performance.md](../nextjs-frontend/16-performance.md) — open only if the task is also about speed.
SCOPE: how an Astro SSR page gets its content and how fresh that content is. The API contract itself is [python-fastapi-backend/12-api.md](../python-fastapi-backend/12-api.md).

The server renders filled-in HTML. The browser gets no content JSON. That is the whole point of this shape: an editor changes the CMS, the visitor reloads, and nothing was rebuilt or redeployed.

---

## Reads

MUST: the page's frontmatter `await`s the API through `src/lib/api.ts`. One call per page, on the server.
MUST: parse into the content type the API declares (`src/lib/content.ts`). MUST NOT: `data as HomeContent` on an unchecked payload — a renamed field then renders as `undefined` in production.
MUST NOT: `fetch` from a browser script to fill a section that the server could have rendered. A skeleton that flashes on every visit is a worse page, not a fresher one.
MUST NOT: `client:load` on a component whose only job is to display server data.

WHEN: the API is down or slow.
MUST: the page still renders. The API serves its last good snapshot or its seed ([python-fastapi-backend/09-modules.md](../python-fastapi-backend/09-modules.md) is where that lives), and the page renders the section's empty state ([nextjs-frontend/01-design.md](../nextjs-frontend/01-design.md) *Four states*) rather than a stack trace.
MUST: a failed fetch is logged on the server with the status. MUST NOT: swallow it silently.
MUST NOT: a hard-coded copy of the content inside a `.astro` file as the fallback. The fallback is the API's job; duplicating it here means two truths and one of them rots.

WHEN: two sections of one page need two endpoints.
HOW: `Promise.all` in the frontmatter. MUST NOT: sequential awaits that make the page as slow as the sum of its parts.

## Freshness

Three caches, and each one must be able to expire and be dropped:

1. **The API's snapshot** — the source-of-truth cache. TTL plus an operator refresh call.
2. **This process's page cache** — optional. A rendered page held for a short TTL so a burst costs one API call. MUST: shorter than the API's TTL. MUST NOT: cache a response that varies per visitor — in this stack nothing does, and the day something does, this cache is the first thing that leaks it.
3. **The visitor's browser** — `Cache-Control` on the HTML. MUST: short (`max-age` in the tens of seconds) with `stale-while-revalidate`. MUST NOT: a long `max-age` on HTML; the reload is how a visitor sees new content.

MUST: send `ETag`/`If-None-Match` through to the API when the client library supports it — an unchanged page should cost a 304, not a re-render.
MUST NOT: a build step that bakes content into HTML. If a change needs `npm run build` to appear, this stack is being used wrong.

## Writes

MUST: a form posts to FastAPI. The Astro server is not a place where writes happen.
MUST: the form works as a form — `method="post"` and an `action`, or `fetch` from a script that is progressive, not the only path.
MUST: show the four states on the form itself: idle, submitting, error (the API's `message`), success.
MUST: the browser posts to the **same origin** (`/api/...` through the proxy), so no CORS and no API URL in the bundle.
MUST NOT [critical]: a secret, an admin token, or a private base URL inside a browser script.
MUST NOT: trust a client-side validity check as the rule. The API validates; the form is a courtesy.

## Server or browser

Server (frontmatter, `.astro` templates): everything that reads content, composes markup, or decides what exists.

Browser (`src/scripts/`): only what needs an event, a timer, or measurement — a rotator, a counter, a flip card, a menu, an intersection observer.

MUST: a browser script is inert until its element exists (`querySelector` guard), and does nothing when the element is absent — pages share scripts.
MUST: `defer`, or a module script. MUST NOT: a blocking `<script>` in `<head>` for a behaviour that can wait.
MUST NOT: read content out of the DOM that the server already had. Pass it through a `data-` attribute or render it — do not re-parse rendered text.
MUST NOT: a global on `window` as the way two scripts talk.

## Images

MUST: design assets that ship with the repo live in `src/assets/` and go through Astro's image pipeline (hashed, `webp`, sized).
MUST: CMS media arrives as a URL from the API and is emitted as a plain `<img>`. It is already mirrored and immutably cached by the backend, and it cannot go through a build-time pipeline in a no-rebuild system.
MUST: every `<img>` has `alt`, `width`, `height` (or a CSS aspect ratio that reserves the box), `loading="lazy"` below the fold, and `decoding="async"`.
MUST NOT: a 3000px original where the element renders at 400px. Ask the API for the rendition that matches the slot.

## Done

- [ ] Content came from the server, in one round of parallel calls
- [ ] The page renders when the API does not answer
- [ ] Nothing content-shaped is fetched from the browser
- [ ] Every cache in the path has a TTL and a way to be dropped
- [ ] Writes go to FastAPI, same origin, with all four states shown
- [ ] Browser scripts guard for their element and ship no secret
- [ ] Images are sized, lazy below the fold, and asked for at the size they render
