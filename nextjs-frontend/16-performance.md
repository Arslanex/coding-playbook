# 16 · Performance

WHEN: a slow page, a large client bundle, images, a waterfall, or "let's add a client cache for speed."
LOAD: this file. Server vs client: 05. Reads: 07.
RELATED: 01 (density, skeletons) · 11 (images via primitives if any) — open only if the task is also that topic.
SCOPE: time to usable UI and JS cost. Not FastAPI pool sizing.

A Client Component is a byte cost. `"use client"` on a layout is a waterfall.

---

## Decide: make it cheaper

Stop at the first yes.

1. The page is a Client Component or a huge `"use client"` layout?
   → Split (05). Server fetch the data (07).
2. Images without `next/image` (or the project's image helper), no width/height?
   → Fix the image. MUST NOT: un-sized blobs that shove the layout (01).
3. Waterfall: client fetch after a spinner, then another fetch?
   → Load on the server in parallel (`Promise.all`). MUST NOT: `useEffect` chain.
4. A chart/editor that is huge?
   → dynamic `import()` on the island that needs it. MUST NOT: pull it into `layout.tsx`.
5. User data cached as static HTML?
   → That is a bug (07), not a speedup.

MUST NOT: add Redis or React Query "for performance" on day one.
MUST NOT: Extra 02-style split of Next apps because a page is slow.

---

## Caps

Bundle: do not ship a date library + a moment clone + a chart on a login page.
Lists: cursor (07), not render 10k rows.
Fonts: one family (+ optional mono) via `next/font`. MUST NOT: three Google families (01).

---

## Done

- [ ] First paint is server data, not a client waterfall
- [ ] Images sized; heavy widgets dynamic
- [ ] `"use client"` still a leaf
- [ ] No cache of private data as static
