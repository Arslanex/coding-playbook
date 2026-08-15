# 02 · Coding principles

WHEN: every TypeScript / React edit, before the first line.
LOAD: this file. Placement: [03-file-structure.md](03-file-structure.md). Visual rules: [01-design.md](01-design.md) if the change is on screen.
RELATED: 10 (feature package) · 11 (`ui/` primitives) — open only if the task is also that topic.
SCOPE: all TS/TSX this agent writes. Not Python.

---

## One job

MUST: a function, component, or hook does one thing. Its name is that thing.

MUST NOT: `OrderCardAndForm`. MUST NOT: a hook named `useOrder` that fetches, mutates, and owns toasts.

SHOULD: a component body you can read without scrolling past unrelated helpers. Split at ~300 lines, or as soon as the file has **two reasons to change** (e.g. markup shape and a product rule living in the same file).

MUST NOT: `utils.ts` / `helpers.ts` / `common.ts`. A helper is named after the work (`formatMoney.ts`) and lives in the feature that owns it, or in `lib/` only if it has no product noun (03).

---

## Types

MUST: `strict` TypeScript. MUST NOT: `any`. MUST NOT: `as` to silence a mismatch — fix the type or parse with Zod (09).

MUST: props typed. A public component has an exported `FooProps` (or the props inline if it has one caller).

MUST NOT: `enum` for API strings — use a Zod enum / union that matches the backend field. MUST NOT: duplicate the backend's Pydantic model by hand in three files; one Zod schema per response the feature reads (09, 10).

---

## Signatures and props

A call you cannot read without opening the definition is a call the next agent gets wrong.

**A plain function or a hook:** MUST: at most **three** parameters. At four, either it does two jobs (split it) or the values travel together (pass one object).

**A component:** it already takes one parameter — the props object. The count that matters is how many **fields** are on it, and the test is not the number but whether they describe one thing.

WHEN: props pass five fields, check. Stop at the first that applies:

1. **They describe two different things** → it is two components. Split.
2. **They are pieces of one record** → pass the record (`order`, not `orderId` + `orderTotal` + `orderStatus` + `orderCreatedAt`).
3. **They genuinely are one thing with many parts** (a form field: label, error, hint, required, id) → fine. Leave it.

MUST NOT: spread an object into props to make the count look smaller (`<Card {...order} />`). The contract disappears and the next rename breaks it silently.

MUST NOT: boolean flag props that select a look — `isLarge`, `isPrimary`, `showBorder`, `compact`. Two booleans are four states and nobody drew three of them. Use one named `variant` / `size` union, so the impossible combinations cannot be typed (11).
Booleans that describe **state** are fine: `disabled`, `isLoading`, `isOpen`.

MUST: `children` for content a parent supplies. MUST NOT: a `title`, `body`, `footer` prop trio where `children` and composition would do.

MUST NOT: a render-prop or a callback that exists only to hand the parent data the parent already has.

---

## What crosses a boundary

MUST: a typed value. MUST NOT: a bare object literal as an informal contract — the shape then lives in the caller's head and nothing checks it.

**Zod schema** — the API boundary only (09). It is what `unknown` becomes on the way in and what the request body is validated against on the way out.
MUST NOT: pass a Zod-inferred **request** type down through the component tree as the display model. The wire shape becomes the UI shape, and the next API version reaches into components that have nothing to do with the API.

**A named type** — everything internal. Derived from the schema (`z.infer`) or declared where the feature owns it.
MUST NOT: `Data`, `Info`, `Params`, `Props2`. Name the thing: `OrderSummary`, `CancelDraft`.

**Dates and times** — the API sends UTC (backend 06). MUST: keep it UTC in state and in the URL; format to local **only** at the moment of display, with `Intl.DateTimeFormat` or the project's date library.
MUST NOT: `new Date(string)` on a date-only value (`2026-08-15`) and then format it — that parses as UTC midnight and renders as the previous day for anyone west of London. Treat a date-only field as a string until something needs a calendar.
MUST NOT: send a locally-formatted string back to the API. It goes back the way it came.
MUST NOT: `toLocaleDateString()` in a Server Component and expect it to match the user — the server's timezone is not theirs. Format in a client leaf, or send both the raw value and let the browser do it (05).

MUST NOT: return a tuple of unrelated values from a hook (`[order, total, warnings]`). The caller destructures by position and a reorder breaks it silently — return a named object.

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
- [ ] Function/hook under four parameters; props describe one thing; no boolean flag prop selecting a look
- [ ] Values crossing a boundary are named types; Zod stays at the API edge
- [ ] No `any` / no silencing `as` / no `utils.ts`
- [ ] Header `Purpose` says what this file must not do
- [ ] No `useEffect` fetch for the initial read
