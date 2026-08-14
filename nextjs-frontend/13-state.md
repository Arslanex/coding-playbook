# 13 · Client state

WHEN: `useState`, URL search params, a global store, or "we need Zustand/Redux/Context."
LOAD: this file. First load is still 07. Interactive leaf: 05.
RELATED: 06 (searchParams) · 08 (after write, refresh) — open only if the task is also that topic.
SCOPE: state that is not the server resource.

---

## Decide

Stop at the first yes.

1. It is in the URL or should be shareable (tab, filter, `orderId`)?
   → `searchParams` / path. MUST NOT: React state as the only copy.
2. It is the resource from FastAPI?
   → Server fetch (07). After a write: refresh. MUST NOT: a client cache as the source of truth on day one.
3. It is UI-only and dies on navigation (modal open, accordion)?
   → local `useState` on the client leaf.
4. It must survive across distant leaves and is still not the resource (theme, sidebar collapsed)?
   → one small context or a store with **that** job. MUST NOT: `useAuthStore` that holds the user and the orders and the toasts.

MUST NOT: Redux/Zustand "because we will have a lot of state." MUST NOT: Context wrapping the whole app for one modal.

Refetch in-place (live table): Extra / a later choice. Default remains server fetch + refresh.

---

## Done

- [ ] Shareable state is in the URL
- [ ] Resources still come from the API
- [ ] No god store
