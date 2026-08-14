# 01 · Playbook boundary

WHEN: first edit in a session, or before touching files under `coding-playbook/`.
LOAD: [playbook root](../README.md) and this file only.
RELATED: [02-turn.md](02-turn.md) — open when executing a user task.
SCOPE: what you may edit and where application code lives.

---

## WHAT this repo is

WHEN: you need to know where to write code.
WHERE: application code lives in the **product repo** (`backend/`, `frontend/`, …) — not inside `coding-playbook/`.
HOW: treat `coding-playbook/` as rules only; implement in the product tree per stack file-structure.

MUST NOT: copy the whole playbook into `src/`.
MUST NOT: scaffold application folders because Extra docs exist.

---

## WHICH stack

WHEN: the user asks for a change and files are named or implied.
WHERE: pick **one** primary stack for this turn.

WHEN: editing Python under the app `backend/` tree (FastAPI HTTP, workers, `src/`).
LOAD: [python-fastapi-backend/README.md](../python-fastapi-backend/README.md).
WHERE: backbone is `config/` · `http/` · `modules/` · `infra/` · `workers/`.

WHEN: editing TypeScript/React/Next under `frontend/`.
LOAD: [nextjs-frontend/README.md](../nextjs-frontend/README.md).
WHERE: backbone is `app/` · `features/` · `ui/` · `lib/`.

MUST NOT: copy FastAPI folders (`http/`, `modules/`, `infra/`, `workers/`) into `frontend/`.
MUST NOT: invent a third stack layout from chat.

WHEN: no playbook stack matches the files being touched.
HOW: stop. Ask the user or add a stack folder — do not guess layout.

---

## HOW to LOAD stack rules

WHEN: after this file and the chosen stack README.
HOW: open **one** numbered topic file for the current task.
HOW: open siblings **only** if that file's `LOAD:` line names them.

MUST NOT: preload all `01`–`16`.
MUST NOT: preload `extra/` unless that shape **already exists** in the application repo.

---

## Extra shapes

WHEN: the **shipped product** already has multi-tenant isolation, SSO, in-product agent runs, search engine, outbox, webhooks, retention, etc.
LOAD: the matching file under `extra/` for that stack — **in addition to** the numbered file for the task, not instead of `01`–`16`.

WHEN: the product does **not** yet have that shape.
MUST NOT: create `modules/agents/`, `app/i18n/`, outbox tables, or `src/agents/` because Extra docs exist.

WHEN: the task is **you** editing code (Cursor, etc.).
HOW: use stack `01`–`16` and [agents/](README.md).

WHEN: the task is **in-product** LLM runs (`AgentRun`, transcript UI, worker agent jobs).
LOAD: [backend extra 03](../python-fastapi-backend/extra/03-agent-teams.md) and/or [frontend extra 08](../nextjs-frontend/extra/08-agents.md) — that is product runtime, not coding-agent routing.

---

## Adapt playbook in git

WHEN: a playbook rule fights a real constraint in **this** product (host, legal, already-shipped tree).
WHERE: the matching playbook numbered or Extra file in `coding-playbook/`.
HOW: change the rule; leave a one-line reason in `SCOPE` or at the decision; then implement in application code.

MUST NOT: silently invent a parallel layout in `src/` while playbook still states the old rule.
MUST NOT: rewrite playbook files for taste when the product does not need a different shape.

---

## Entry paths (first numbered files)

WHEN: user starts a **new FastAPI service** and no topic file is obvious yet.
LOAD: backend `01-coding-principles` → `02-file-structure` → `03-config`, then the topic file.

WHEN: user starts a **new Next.js app** and no topic file is obvious yet.
LOAD: frontend `01-design` → `02-coding-principles` → `03-file-structure`, then the topic file.

WHEN: user asks for **security or PR review**.
LOAD: stack `15-security`; add `14-testing` when tests are in scope.

WHEN: user adds **SSO or tenant** after the feature already ships.
LOAD: matching `extra/` file for that stack — not before the feature exists.

---

## Done

MUST: before writing application code, confirm:

- Application paths are separate from `coding-playbook/`
- One stack is chosen; the other stack is untouched this turn
- Numbered LOAD set is minimal (not all 16)
