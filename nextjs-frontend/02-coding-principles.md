# 02 · Coding principles

WHEN: every TypeScript / React edit, before the first line.
LOAD: this file. Placement: [03-file-structure.md](03-file-structure.md). Visual rules: [01-design.md](01-design.md) if the change is on screen.
RELATED: 10 (feature package) · 11 (`ui/` primitives) — open only if the task is also that topic.
SCOPE: all TS/TSX this agent writes. Not Python.

---

## One job

MUST: a function, component, or hook does one thing. Its name is that thing.

MUST NOT: `OrderCardAndForm`. MUST NOT: a hook named `useOrder` that fetches, mutates, and owns toasts.

SHOULD: a component body you can read without scrolling past unrelated helpers. Split when 01's file-split idea applies (~300 lines, two reasons to change).

MUST NOT: `utils.ts` / `helpers.ts` / `common.ts`. A helper is named after the work (`formatMoney.ts`) and lives in the feature that owns it, or in `lib/` only if it has no product noun (03).

---

## Types

MUST: `strict` TypeScript. MUST NOT: `any`. MUST NOT: `as` to silence a mismatch — fix the type or parse with Zod (09).

MUST: props typed. A public component has an exported `FooProps` (or the props inline if it has one caller).

MUST NOT: `enum` for API strings — use a Zod enum / union that matches the backend field. MUST NOT: duplicate the backend's Pydantic model by hand in three files; one Zod schema per response the feature reads (09, 10).

---

## File header

MUST: first lines of every `.ts` / `.tsx` that is not a generated or Next special file (`page.tsx` may be short — still a one-line `Purpose` comment if it does more than compose).

```tsx
/**
 * Feature: orders
 * Kind: Page | Panel | Field | Schema | Api | Primitive | Lib | Test
 * Purpose: List the caller's orders. Does not cancel.
 */
```

MUST: English. MUST NOT: a component inventory in `Purpose`.

---

## Hooks and effects

MUST: a hook `use*` is for a real React need (state that the server cannot send, a browser API). MUST NOT: `useEffect` + `fetch` for the first load (07).

MUST NOT: `useState` for data the URL already has (`page`, `orderId`, tab). That is the URL (06, 13).

---

## Imports

MUST: feature A does not import feature B's inner components. It imports a page-level export or goes through the public panel B already exported (10).

MUST NOT: import from `ui/` a product noun (`ui/OrderCard`). That is a feature.

---

## Done

- [ ] One job per function/component/hook
- [ ] No `any` / no silencing `as` / no `utils.ts`
- [ ] Header `Purpose` says what this file must not do
- [ ] No `useEffect` fetch for the initial read
