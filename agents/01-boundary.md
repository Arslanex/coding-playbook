# 01 · Playbook boundary

WHEN: first edit in a session, or before touching files under `coding-playbook/`.
LOAD: [AGENTS.md](../AGENTS.md) and this file only.
RELATED: [02-turn.md](02-turn.md) — open when executing a user task.
SCOPE: what you may edit, what you may run, and where application code lives.

---

## WHAT this repo is

WHEN: you need to know where to write code.
WHERE: application code lives in the **product repo** (`backend/`, `frontend/`, …) — not inside `coding-playbook/`.
HOW: treat `coding-playbook/` as rules only; implement in the product tree per stack file-structure.

MUST NOT: copy the whole playbook into `src/`.
MUST NOT: scaffold application folders because Extra docs exist.

---

## WHAT you may run

Your job is the working tree. Anything that **records, publishes, or deploys** belongs to the user, and you do it only when they ask for it in this conversation.

WHEN: the action would leave the working tree or be hard to undo.
MUST NOT: do it unless the user asked. MUST NOT: do it and mention it afterwards.

- **git writes** — `commit`, `push`, `branch`, `checkout -b`, `merge`, `rebase`, `reset`, `revert`, `stash`, `tag`, `gh pr create`. A commit rewrites work the user was still composing; a push is public and may trigger CI.
- **Containers and deployment** — a `Dockerfile`, a compose file, `docker build` / `run`, CI workflow files, cloud or IaC config, anything that publishes to a registry.
- **Migrations against a real database** — writing the revision is your job (07); running `alembic upgrade head` against anything but a local or test database is not.
- **The machine outside this project** — global installs, changing shell config, touching files above the repo root.

WHEN: always allowed, no permission needed.

- **Read-only git** — `status`, `diff`, `log`, `show`, `blame`. These are how you find out what actually changed; not using them is the bigger failure ([07-verify.md](07-verify.md)).
- Editing, creating, and deleting files inside the product tree that the task names
- The project's own local commands: its tests, linter, type checker, dev server, local migration against the test database

WHEN: the work genuinely needs one of the restricted actions — the change is not testable without a compose service, or the user's goal implies a PR.
HOW: say what is needed and why in one line, give the exact command, and stop. The user runs it, or tells you to.

WHEN: the user has already asked for one of these in this conversation (`"commit this"`, `"open a PR"`).
HOW: do it. Their instruction stands for what they asked for — not for everything after it. `"commit this"` is not standing permission to push.

MUST NOT: read this as a reason to leave work unfinished. Write the file, run the tests, report the result — then stop at the line above.

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

These two stacks are the whole map: an HTTP API and a web UI. Anything else — mobile, CLI, desktop, embedded, a data pipeline, an ML training job, a bot — has **no rules here**.

WHEN: the user **describes** work outside these two, at any point.
HOW: say so before writing anything. Two ways forward: leave it out of scope, or add a stack folder and write its map ([AGENTS.md](../AGENTS.md), *Stacks*). Question C of [08-architecture.md](08-architecture.md) exists to catch this at the start.
MUST NOT: wait until you are touching the file to notice. By then the user has already been told it would be built.
MUST NOT: bend a stack onto it. A CLI is not a FastAPI app with the routes removed; a data pipeline is not a worker with no queue.

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

WHEN: the user describes a **product** rather than a change — an empty repo, "build me an X", no data model agreed yet.
LOAD: [08-architecture.md](08-architecture.md) **before** any stack file. Nothing gets written until its six questions are answered, the synthesis is confirmed, and the data model exists.

WHEN: user starts a **new FastAPI service** and the architecture and data model are already agreed.
LOAD: backend `01-coding-principles` → `02-file-structure` → `03-config`, then the topic file. Build order: [08-architecture.md](08-architecture.md).

WHEN: user starts a **new Next.js app** and the API it consumes already returns real data.
LOAD: frontend `01-design` → `02-coding-principles` → `03-file-structure`, then the topic file.
MUST NOT: build the UI first against an endpoint that does not exist ([08-architecture.md](08-architecture.md) step 6).

WHEN: user asks for **security or PR review**.
LOAD: stack `15-security`; add `14-testing` when tests are in scope.

WHEN: user adds **SSO or tenant** after the feature already ships.
LOAD: matching `extra/` file for that stack — not before the feature exists.

WHEN: creating or updating product `README.md` / `docs/`, or the change moves architecture, API, or how the app is run, **or** the task is a ship / handover / close.
LOAD: [09-docs.md](09-docs.md). Application `docs/`, not this playbook.

---

## Done

MUST: before writing application code, confirm:

- Application paths are separate from `coding-playbook/`
- One stack is chosen; the other stack is untouched this turn
- Everything the user described is covered by a stack that exists — anything else was raised, not silently reshaped
- Numbered LOAD set is minimal (not all 16)
- Nothing was committed, pushed, containerised, or deployed without being asked
