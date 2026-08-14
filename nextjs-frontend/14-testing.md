# 14 · Tests

WHEN: adding or changing a test, a testing-library render, or a Playwright spec.
LOAD: this file. What the UI must look like: 01. What the API returns: 09.
RELATED: 02 (header) · 10 (feature under test) — open only if the task is also that topic.
SCOPE: `frontend/tests/`. Not a dump inside `app/`.

---

## Where

```
features/orders/OrderList.tsx  →  tests/features/orders/OrderList.test.tsx
ui/Button.tsx                  →  tests/ui/Button.test.tsx
lib/api.ts                     →  tests/lib/api.test.ts
```

MUST NOT: `tests/unit/` vs `e2e/` as the only folders. Markers (`e2e`) are extra; the path still mirrors.

---

## What to test

Feature panel: four states (01) with **mocked** `api.ts` — loading/empty/error/list. One happy action. MUST NOT: assert on hex or class strings that are tokens.

`lib/api.ts`: maps a FastAPI error body to `ApiError`.

E2E (Playwright) for the login + one critical write, against a test API or MSW. MUST NOT: E2E for every badge.

MUST NOT: a test that only snapshots a whole page to lock AI-slop in.

---

## Done

- [ ] Path mirrors `features/` / `ui/` / `lib/`
- [ ] Four states covered for a list/detail that shipped
- [ ] `ApiError` mapping tested
