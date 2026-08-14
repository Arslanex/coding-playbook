# 02 · Turn discipline

WHEN: every user message that asks for code changes, fixes, or reviews.
LOAD: this file and [01-boundary.md](01-boundary.md) if not already open. Then stack README + **one** numbered topic file (+ that file's `LOAD:` siblings only).
RELATED: [03-anti-patterns.md](03-anti-patterns.md) when scope is unclear · [04-errors.md](04-errors.md) when user pasted an error.
SCOPE: one turn — WHEN to act, WHERE to edit, HOW to proceed.

---

## Parse the user message (before any edit)

WHEN: user message arrives.
HOW: extract all of the following; if any is missing on a vague prompt, ask **one** question or state one-line assumptions — MUST NOT invent a full product.

- **Stack** — `python-fastapi-backend` or `nextjs-frontend` (one primary stack per turn unless user explicitly splits)
- **Task** — one sentence describing the change
- **Application files** — paths in the product repo, not playbook
- **Done when** — observable checklist (tests pass, env field names, no new folders)

MUST NOT: proceed on "build a dashboard", "set up config", or "fix my app" without stack, paths, entities, or verbatim error text.

---

## One turn, one slice

WHEN: executing the parsed task.
WHERE: only files named in the task or required by the loaded numbered playbook file.
HOW: smallest diff that satisfies done-when.

MUST NOT: refactor unrelated files "while here".
MUST NOT: implement an entire feature chain in one turn unless the user explicitly ordered that chain.

WHEN: user asks for a **full feature** across multiple turns.
HOW: use this order — one turn per step unless user combines steps:

1. Model + migration — LOAD backend `06-database`, `07-migrations`
2. Service + HTTP — LOAD backend `09-modules`, `10-http`, `12-api`
3. Worker if needed — LOAD backend `11-workers` + owning module
4. UI feature — LOAD frontend `01-design`, `03-file-structure`, `10-features`, `07-data`
5. Tests — LOAD stack `14-testing`

WHEN: task touches **backend config** and **frontend env** in the same message.
HOW: ask user to split into two turns — backend `03-config` first, frontend `04-config` second.
MUST NOT: mix both stacks' config in one diff unless user explicitly requires both.

---

## LOAD by task type

WHEN: task is **config or env** (new env var, limit, timeout, secret name).
WHERE (backend): `src/config/settings.py`, `.env.example` — LOAD [03-config](../python-fastapi-backend/03-config.md).
WHERE (frontend): `frontend/lib/env.ts`, `.env.example` — LOAD [04-config](../nextjs-frontend/04-config.md).

WHEN: task is **new API endpoint**.
WHERE: `modules/<noun>/`, mount via `http/router.py` only.
LOAD: backend `09-modules`, `10-http`, `12-api`.

WHEN: task is **DB column or table change**.
WHERE: model in `infra/db`, migration in Alembic — same turn.
LOAD: backend `06-database`, `07-migrations`.

WHEN: task is **background job** (email, export, agent step, long work).
WHERE: `workers/jobs/`, queue publish from service — not HTTP, not `BackgroundTasks`.
LOAD: backend `11-workers` + owning module `09-modules`.

WHEN: task is **new screen or UI feature**.
WHERE: `features/<noun>/` — not a city under `app/`.
LOAD: frontend `01-design`, `03-file-structure`, `10-features`; add `07-data` for reads, `08-forms` for writes.

WHEN: task is **auth or session**.
WHERE (backend): identity, cookies, JWT, authz in services — LOAD `13-identity-security`.
WHERE (frontend): HttpOnly cookie session — LOAD `12-auth`.
HOW: prefer **separate turns** per stack unless user explicitly combines.

WHEN: task is **security or PR review**.
LOAD: stack `15-security`; add `14-testing` if tests are in scope.

WHEN: task is **pool, timeout, cap, or performance**.
LOAD: backend `16-performance` and `03-config` (every cap is a Settings field).

MUST NOT: open a file's `RELATED:` line unless the task is **also** that topic.

---

## Config turns (repeat failure — follow exactly)

WHEN: adding or reading env vars, limits, timeouts, pool sizes, secret **names**.

WHERE (backend): `src/config/` only for `os.getenv`; inject `get_settings()` everywhere else.
HOW: add field to `Settings`; update `.env.example` with name only; no magic numbers in modules/infra/workers.
LOAD: [03-config](../python-fastapi-backend/03-config.md).
MUST NOT: `config/constants.py` for product policy — that belongs in `modules/`.

WHERE (frontend): parse once in `lib/env.ts`; export `serverEnv` / `publicEnv`.
HOW: server secrets stay off `NEXT_PUBLIC_*`; features import env only from `lib/env.ts`.
LOAD: [04-config](../nextjs-frontend/04-config.md).
MUST NOT: `process.env` in feature or component files.

---

## Output discipline

WHEN: replying after edits.
HOW: implement; summarize what changed and done-when status.
MUST NOT: paste the whole playbook back.
MUST NOT: restate every MUST from loaded files unless user asked for a summary.

WHEN: adapting a playbook rule for this product.
WHERE: matching playbook file in `coding-playbook/`.
HOW: edit playbook with one-line reason in same change set (or tell user to commit it).

---

## User prompt shape

WHEN: user message contains `Stack:`, `Task:`, `Files:`, `Done when:` blocks.
HOW: obey literally; MUST NOT expand beyond `Constraints:`.

---

## Done (self-check before stop)

MUST verify:

- Only application files in scope were edited
- Loaded playbook set was minimal
- No new `utils/`, `helpers/`, or parallel service tree
- No Extra scaffold without existing product shape
- Config/env rules satisfied if env was touched
- User's done-when items met or explicitly listed as blocked

MUST NOT: claim done without running or citing tests when user required them.
