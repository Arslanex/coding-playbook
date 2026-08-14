# 08 · Forms and writes

WHEN: a `<form>`, a mutation, a Server Action, or "save / cancel / delete" on the client.
LOAD: this file. Design of the form: [01-design.md](01-design.md). API: 09. Authz is still FastAPI (12).
RELATED: 05 (which leaf is client) · 07 (refresh after write) — open only if the task is also that topic.
SCOPE: how this app **sends** changes. The rule "may cancel if unpaid" stays on the API.

---

## Where the write goes

MUST: POST/PATCH/DELETE FastAPI `/v1/…`. Next is not a second backend.

Server Actions: allowed as a **thin** proxy (`"use server"` function that calls `lib/api.ts` and `revalidatePath` / `redirect`). MUST NOT: a Server Action that contains the business rule, talks to Postgres, or invents a second authz.

MUST NOT: `fetch` from a Client Component as the only write path when the form can be a native `<form action={…}>` — progressive enhancement until 05 forces a client island (autosave, rich widget).

---

## Schema

MUST: Zod for the request body the UI sends, aligned with the API request schema. Shared types: duplicate the fields this screen has, not the whole backend dump.

`extra` fields the API forbids must not be in the form. MUST NOT: hidden `user_id` / `role` the client invents (15).

---

## UX

One primary submit (01). Errors: map `error_code` to a field when `details.fields` exists; otherwise the form banner uses `message`. MUST NOT: `alert(JSON.stringify(err))`.

Disabled submit while in flight. MUST NOT: double POST without an idempotency key when the API requires one (backend 09).

Destructive: noun in the button, confirm (01).

After success: `revalidatePath` / `router.refresh()` so the server read (07) is fresh. MUST NOT: patch local state as the only truth.

---

## Done

- [ ] Write hits FastAPI; no rule in the Action
- [ ] Zod on the way out; API `error_code` on the way in
- [ ] One primary; field errors; refresh after success
