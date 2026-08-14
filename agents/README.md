# Agent operations map

WHEN: you are a **coding agent** (Cursor, Claude Code, Copilot, …) editing application code with this playbook — not an in-product LLM job ([python-fastapi-backend/extra/03-agent-teams.md](../python-fastapi-backend/extra/03-agent-teams.md)).
LOAD: [playbook root](../README.md) first. Then this map. Then only the files named in the matching WHEN block below.
SCOPE: routing for coding agents. Human prose at repo root is for users — MUST NOT load unless the user `@`-references it.

MUST NOT: load every file in `agents/` for one edit.
MUST NOT: load all stack `01`–`16` files for one edit.
MUST NOT: copy `coding-playbook/` into application `src/`.

---

## Default LOAD chain (every session)

WHEN: any code task begins.
HOW: load in this order — stop adding files once the turn's topic file is open.

1. [../README.md](../README.md) — root rules, stack list, Extra gates, adapt-in-git
2. [01-boundary.md](01-boundary.md) — once per session (playbook vs app, which stack)
3. [02-turn.md](02-turn.md) — every turn (parse task, one slice, which numbered file)
4. Stack README — [python-fastapi-backend](../python-fastapi-backend/README.md) **or** [nextjs-frontend](../nextjs-frontend/README.md)
5. **One** numbered `01`–`16` file for the task (+ that file's `LOAD:` siblings only)

---

## WHEN → which agent file

WHEN: **first code edit in the session** or you do not yet know playbook vs application paths.
LOAD: [01-boundary.md](01-boundary.md).

WHEN: **every user message** that changes code, reviews a diff, or fixes a bug.
LOAD: [02-turn.md](02-turn.md).

WHEN: the prompt is **vague** ("build a dashboard", "fix config"), **multi-feature**, or you are about to **create many files**.
LOAD: [03-anti-patterns.md](03-anti-patterns.md) in addition to `02-turn`.

WHEN: the user pasted **error text**, a **stack trace**, an **HTTP failure**, or a **CI/build log**.
LOAD: [04-errors.md](04-errors.md) → match first WHEN block inside it → LOAD the stack file its block names.

WHEN: the user only asks **where the human guide is** (onboarding, prompt tips).
HOW: point to [USER-GUIDE.md](../USER-GUIDE.md) or [HOW-TO-PROMPT.md](../HOW-TO-PROMPT.md).
MUST NOT: load those files unless `@`-referenced.

---

## WHEN → which stack README

WHEN: editing Python under the product `backend/` tree (FastAPI HTTP, workers, Alembic, `src/config`, …).
LOAD: [python-fastapi-backend/README.md](../python-fastapi-backend/README.md).

WHEN: editing TypeScript/React/Next under `frontend/`.
LOAD: [nextjs-frontend/README.md](../nextjs-frontend/README.md).

WHEN: the task names files in **both** stacks in one message.
HOW: ask to split turns — backend turn first unless user explicitly combines ([02-turn.md](02-turn.md)).

WHEN: files match **no** playbook stack.
HOW: stop; do not invent layout ([01-boundary.md](01-boundary.md)).

---

## WHEN → which numbered stack file (pick one primary)

Use [02-turn.md](02-turn.md) for full detail. Quick routing:

WHEN: env var, limit, timeout, secret **name**, pool size.
WHERE: backend → `03-config` · frontend → `04-config`.

WHEN: new log line, logger, handler.
LOAD: backend `04-logging`.

WHEN: exception class, HTTP error JSON, `error_code`.
LOAD: backend `05-errors` (+ `12-api` if public URL shape).

WHEN: model, repository, session, query.
LOAD: backend `06-database`.

WHEN: Alembic revision, column/table change.
LOAD: backend `06-database` + `07-migrations` **same turn**.

WHEN: new infra folder (cache, queue, storage client).
LOAD: backend `08-infra` (+ `03-config` for settings).

WHEN: new module, service, router in `modules/`.
LOAD: backend `09-modules` (+ `10-http` if mounting route).

WHEN: mount list, middleware, deps — not business rules.
LOAD: backend `10-http`.

WHEN: background job, queue, worker, retry/DLQ.
LOAD: backend `11-workers` (+ owning module `09`).

WHEN: public URL, list page, success JSON, status codes.
LOAD: backend `12-api`.

WHEN: JWT, login, refresh, authz, Redis identity keys.
LOAD: backend `13-identity-security` (+ `15-security`).

WHEN: tests, mirror path, fixtures.
LOAD: stack `14-testing`.

WHEN: PR pass, XSS, uploads, webhooks, secrets checklist.
LOAD: stack `15-security`.

WHEN: pool/timeout/cap tuning, slow query, cache decision.
LOAD: backend `16-performance` + `03-config`.

WHEN: new screen, layout, colour, four UI states.
LOAD: frontend `01-design` **before** other frontend files.

WHEN: split file, naming, no `utils/`.
LOAD: frontend `02-coding-principles`.

WHEN: where file lives (`app/` vs `features/` vs `ui/` vs `lib/`).
LOAD: frontend `03-file-structure`.

WHEN: `"use client"`, hooks, server vs client leaf.
LOAD: frontend `05-server-client`.

WHEN: URL, loading.tsx, metadata, `noindex`.
LOAD: frontend `06-routing`.

WHEN: server fetch, RSC data, no client GET waterfall.
LOAD: frontend `07-data`.

WHEN: form, mutation, Server Action shell.
LOAD: frontend `08-forms` (+ `09-api-client`).

WHEN: fetch wrapper, `/v1`, cookies, `error_code`.
LOAD: frontend `09-api-client`.

WHEN: new feature package under `features/<noun>/`.
LOAD: frontend `10-features`.

WHEN: Button, modal, token, primitive.
LOAD: frontend `11-ui` (+ `01-design`).

WHEN: session cookie, protected route, sign-in UI.
LOAD: frontend `12-auth`.

WHEN: URL state, store, search params.
LOAD: frontend `13-state`.

WHEN: RSC, images, bundle, client waterfall.
LOAD: frontend `16-performance`.

MUST NOT: open a file's `RELATED:` line unless the task is **also** that topic.

---

## WHEN → Extra (only if shape already in product)

WHEN: the **shipped application** already has the shape — not because the doc exists.
LOAD: matching `extra/NN-….md` for that stack (+ numbered siblings named inside that Extra file).

WHEN: shape does **not** exist in the product repo yet.
MUST NOT: load or scaffold Extra.

Backend Extra index: [python-fastapi-backend/extra/README.md](../python-fastapi-backend/extra/README.md).
Frontend Extra index: [nextjs-frontend/extra/README.md](../nextjs-frontend/extra/README.md).

---

## Files in this folder

1. [01-boundary.md](01-boundary.md) — playbook vs app; stack choice; Extra gate; adapt in git
2. [02-turn.md](02-turn.md) — parse turn; WHERE to edit; HOW to LOAD; config; done check
3. [03-anti-patterns.md](03-anti-patterns.md) — MUST NOT behaviors
4. [04-errors.md](04-errors.md) — fifty errors as WHEN/WHERE/HOW blocks (A01–E50)

Human mirrors (users): [USER-GUIDE.md](../USER-GUIDE.md) · [HOW-TO-PROMPT.md](../HOW-TO-PROMPT.md) · [VIBE-CODING-PITFALLS.md](../VIBE-CODING-PITFALLS.md) · [VIBE-CODING-ERRORS.md](../VIBE-CODING-ERRORS.md).

---

## Done (before writing code)

MUST: confirm LOAD set matches the situation above — not "everything".
