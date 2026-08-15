# Agent operations map

WHEN: you are a **coding agent** (Cursor, Claude Code, Copilot, …) editing application code with this playbook — not an in-product LLM job ([python-fastapi-backend/extra/03-agent-teams.md](../python-fastapi-backend/extra/03-agent-teams.md)).
LOAD: [AGENTS.md](../AGENTS.md) first. Then this map. Then only the files named in the matching WHEN block below.
SCOPE: routing for coding agents. Human prose lives in `for-humans/` — MUST NOT load it unless the user `@`-references that exact file.

MUST NOT: load every file in `agents/` for one edit.
MUST NOT: load all stack `01`–`16` files for one edit.
MUST NOT: copy `coding-playbook/` into application `src/`.

---

## Default LOAD chain (every session)

WHEN: any code task begins.
HOW: load in this order — stop adding files once the turn's topic file is open.

1. [../AGENTS.md](../AGENTS.md) — entry point: rule strength, stack list, Extra gates, adapt-in-git
2. [01-boundary.md](01-boundary.md) — once per session (playbook vs app, which stack)
3. [08-architecture.md](08-architecture.md) — **only on an empty or new project**. Nothing below runs until its six questions are answered and the data model exists
4. [06-plan.md](06-plan.md) — **first**, if the task already has a plan. You read it; you do not recall it
5. [02-turn.md](02-turn.md) — every turn (parse task, one slice, which numbered file)
6. [05-understand.md](05-understand.md) — once per task, when the code it touches is unfamiliar
7. Stack README — [python-fastapi-backend](../python-fastapi-backend/README.md) **or** [nextjs-frontend](../nextjs-frontend/README.md)
8. **One** numbered `01`–`16` file for the task (+ that file's `LOAD:` siblings only)
9. [07-verify.md](07-verify.md) — every turn, before replying

Steps 4 and 9 bracket the turn: state comes off disk at the start, evidence goes to the user at the end. Neither is optional on a task that spans turns.

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

WHEN: the user describes a **product**, not a change — empty repo, "build me an X", no data model agreed.
LOAD: [08-architecture.md](08-architecture.md) **before any stack file**. Ask its six questions, produce the data model and architecture, then build in its order.
MUST NOT: write a file before all six have answers.
MUST NOT: assume the work fits this playbook — question C asks that, and a mobile app or CLI has no stack here.

WHEN: the code the task touches is **unfamiliar**, a **new noun** is being added, or you are about to assume something you have not checked.
LOAD: [05-understand.md](05-understand.md). Once per task, before the first edit.

WHEN: the task needs **more than one file**, **more than one turn**, or touches a **migration, auth, or a dependency**. Also every later turn of such a task.
LOAD: [06-plan.md](06-plan.md).
MUST: read the plan before any other file. MUST NOT: keep the plan in the conversation.

WHEN: **before every reply** — and the moment you are about to say "done", "fixed", or "should work".
LOAD: [07-verify.md](07-verify.md).
MUST NOT: report a result without naming what proves it.

WHEN: about to write a package name into `package.json` / `pyproject.toml`, or touch a lockfile.
LOAD: [03-anti-patterns.md](03-anti-patterns.md) Dependencies **before** the manifest edit.
MUST NOT: write a package name into a manifest before confirming it exists.

WHEN: the user only asks **where the human guide is** (onboarding, prompt tips).
HOW: point to [`for-humans/`](../for-humans/README.md) — 01 start here, 02 prompts, 03 reviewing your PRs, 06 glossary.
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

## WHEN → which numbered stack file

Not here. Each stack README carries its own `WHEN → file` table and is the only copy that is kept current:

- [python-fastapi-backend/README.md](../python-fastapi-backend/README.md)
- [nextjs-frontend/README.md](../nextjs-frontend/README.md)

HOW: open the stack README for the turn, match its WHEN line, load **one** numbered file (+ that file's `LOAD:` siblings only).
Multi-file combinations for a feature chain (model + migration, service + HTTP + API): [02-turn.md](02-turn.md) *LOAD by task type*.

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
2. [02-turn.md](02-turn.md) — parse turn; WHERE to edit; HOW to LOAD; one slice; scope check
3. [03-anti-patterns.md](03-anti-patterns.md) — MUST NOT behaviors
4. [04-errors.md](04-errors.md) — fifty errors as WHEN/WHERE/HOW blocks (A01–E50)
5. [05-understand.md](05-understand.md) — read before write; analyse; check vs assume vs ask; design threshold
6. [06-plan.md](06-plan.md) — the plan file; read it first; reconcile before acting; one slice at a time
7. [07-verify.md](07-verify.md) — order of authority; evidence; stop conditions; the reply
8. [08-architecture.md](08-architecture.md) — new project: the six questions, stack coverage, the two documents, build order, when to revise

`01`–`04` say where to look. `05`–`07` say how to work: what you establish before building, what you write down so it survives, and what you must prove before you claim it. `08` says what to build first, and it runs before all of them on an empty repo.

MUST NOT: load all eight for one edit. The Default LOAD chain above is the order; stop when the turn's topic file is open.

Human mirrors (users): [`for-humans/`](../for-humans/README.md) — EN and TR. MUST NOT: load unless `@`-referenced.

---

## Done (before writing code)

MUST: confirm LOAD set matches the situation above — not "everything".
