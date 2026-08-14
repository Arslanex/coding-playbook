# Extra 06 · Search (UI)

WHEN: a search box / results page, not a filtered list (07 cursor list).
LOAD: this file **and** 07, 09, 10. Backend: [python-fastapi-backend/extra/08-search.md](../../python-fastapi-backend/extra/08-search.md).
SCOPE: `features/<noun>/` search panel or `features/search` if search is the product.

Query `q` in the URL (13). Server fetch the search endpoint (07). Parse hits with Zod. Four states (01).

MUST NOT: client-side filter of a downloaded dump as "search." MUST NOT: show a hit the detail URL would 404. MUST NOT: put raw engine DSL in the box (backend Extra 08).

Debounce is a client leaf on the input; submit still navigates to `?q=` so the result is shareable.

---

## Done

- [ ] `q` in the URL; server fetch; Zod hits
- [ ] Empty / error / list; no forbidden snippets
