# AGENTS.md — start here if you are an agent

WHEN: you are an AI coding agent (Cursor, Claude Code, Copilot, Codex, …) about to create or edit application code in a repo that carries this playbook.
LOAD: this file first, in full. Then follow the **Agent LOAD map** below. Nothing else at the repo root is for you.
SCOPE: routing. This file says which file answers which situation; the rules themselves live in the stack folders.

MUST NOT: load every stack. MUST NOT: load all sixteen numbered files. MUST NOT: load anything under `for-humans/` unless the user `@`-references that exact file — it is prose for people and carries no rule you need.

Human readers: you want [`README.md`](README.md) instead.

---

## What this folder is

This folder is playbook, not application source. MUST NOT: copy it into `src/`.

Your job is the working tree. MUST NOT: commit, push, open a PR, write a `Dockerfile` or CI workflow, or deploy anything unless the user asked for it in this conversation — name the command and let them run it ([agents/01-boundary.md](agents/01-boundary.md) *WHAT you may run*). Read-only git (`status`, `diff`, `log`) is always fine and you should use it.

---

## Rule strength

Three levels. A rule's level decides what you do when the user asks for the opposite.

**`MUST [critical]` / `MUST NOT [critical]`** — breaking it leaks a secret, loses or exposes another user's data, or removes an authorization gate. Not a matter of taste.
HOW: do not write the code. Say which rule and what it exposes, in one or two sentences, and offer the safe shape. If the user still wants it after that, it is their call — do it, and note in the same message what is now exposed. MUST NOT: an argument that spans several turns.

**`MUST` / `MUST NOT`** (unmarked) — the shape this playbook decided on. Consistency, not safety.
HOW: follow it by default. If the user asks for something else, say the rule exists in one clause and do what they asked. A shipped product that already contradicts it wins — then write the adaptation into the file (*Not carved in stone* below).

**`SHOULD` / `SHOULD NOT`** — a default worth keeping. Deviate when the task has a reason; no discussion needed.

MUST NOT: treat an unmarked `MUST NOT` as a safety rule and refuse a user's request over it.
MUST NOT: add `[critical]` to a new rule unless a secret, another user's data, or an authz gate is on the line.

`[critical]` markers are applied in both stacks. `agents/` files are unmarked: they are process rules, and the strength that matters there is the stack rule they point at.

---

## Agent LOAD map (which file, which situation)

WHEN: **first code edit in a session** (or stack unknown).
LOAD: [agents/01-boundary.md](agents/01-boundary.md).

WHEN: **every user message** that changes code, reviews a diff, or fixes a bug.
LOAD: [agents/02-turn.md](agents/02-turn.md).

WHEN: prompt is **vague**, **multi-feature**, or you are about to **create many files / load many docs**.
LOAD: [agents/03-anti-patterns.md](agents/03-anti-patterns.md).

WHEN: user pasted **error text**, stack trace, HTTP status failure, or CI log.
LOAD: [agents/04-errors.md](agents/04-errors.md) → then the stack numbered file named in the matched block.

WHEN: the user describes a **product** rather than a change — empty repo, no agreed data model.
LOAD: [agents/08-architecture.md](agents/08-architecture.md) **first**. Six questions, then a data model and an architecture the user has seen, then build from the core outward.

WHEN: creating or updating product `README.md` / `docs/`, a code change that moves architecture, data model, API, authz, setup, or deploy, a decision that must outlive this task, **or** a ship / handover / project close.
LOAD: [agents/09-docs.md](agents/09-docs.md). Task notes stay in the plan ([agents/06-plan.md](agents/06-plan.md)); surviving product truth stays in `docs/`.

WHEN: the code is **unfamiliar**, a **new noun** is being added, or you are about to assume something you did not check.
LOAD: [agents/05-understand.md](agents/05-understand.md) — once per task, before the first edit.

WHEN: the task needs **more than one file or more than one turn**, or touches a migration, auth, or a dependency — and every later turn of it.
LOAD: [agents/06-plan.md](agents/06-plan.md) **before any other file**. The plan lives on disk, not in the conversation.

WHEN: **before every reply**, and whenever you are about to say "done" or "should work".
LOAD: [agents/07-verify.md](agents/07-verify.md).

WHEN: editing **Python** under `backend/` (FastAPI, workers).
LOAD: [python-fastapi-backend/README.md](python-fastapi-backend/README.md) → **one** numbered `01`–`16` file for the task (+ its `LOAD:` siblings only).

WHEN: editing **TypeScript/React/Next** under `frontend/`.
LOAD: [nextjs-frontend/README.md](nextjs-frontend/README.md) → **one** numbered `01`–`16` file for the task (+ its `LOAD:` siblings only). Visual work: frontend `01-design` before other frontend files.

WHEN: product **already ships** an Extra shape (tenant, SSO, in-product agents, search, …).
LOAD: matching `extra/NN-….md` for that stack **in addition to** the numbered file — not instead of `01`–`16`.

WHEN: playbook rule must change to match **this** product (see Not carved in stone).
WHERE: the matching numbered or Extra playbook file.
HOW: edit playbook with one-line reason in `SCOPE`; then implement in application code.

WHEN: user asks for human onboarding or prompt templates.
HOW: point to [`for-humans/`](for-humans/README.md) — 01 to adopt the playbook, 02 for prompts, 03 to review a PR. MUST NOT: load them yourself unless `@`-referenced.

Full agent routing index: [agents/README.md](agents/README.md).

MUST NOT: load all of `agents/` for one edit.
MUST NOT: load all sixteen numbered files for one edit.
MUST NOT: load Extra because it might be useful later.

---

## Not carved in stone

These files are a **starting map**, not scripture. They must fit **this** project.

MUST: if a rule here fights a real product constraint (stack, host, legal, an already-shipped shape), **adapt the rule to the project** — then **write that adaptation into the matching playbook file** in this folder (numbered file or Extra). The next agent reads the docs, not the chat.

MUST NOT: silently ignore a file and invent a parallel layout in `src/` while this folder still says the old thing.
MUST NOT: rewrite a file "for taste" when the project does not need a different shape.
MUST NOT: move Extra into day-one 01–16 because it might be useful later. Extra still loads only when that shape **already** exists.

How to change a playbook file: keep `WHEN` / `LOAD` / `MUST` / `MUST NOT`. Change the rule, the tree, or the Extra trigger. Leave a one-line reason in the file's `SCOPE` or at the Decide that moved. Do not leave two conflicting sentences.

---

## Stacks

1. Python + FastAPI API / workers — [python-fastapi-backend/README.md](python-fastapi-backend/README.md)
2. Next.js UI — [nextjs-frontend/README.md](nextjs-frontend/README.md)

No matching stack: stop. Do not invent a parallel layout from another stack. If this repo needs a new stack, add a folder and one line here — then write its map the same way.

## Extra (Python FastAPI)

Only if that API **already** has the shape. Map: [python-fastapi-backend/README.md](python-fastapi-backend/README.md) Extra section.

1. Isolated customers — [python-fastapi-backend/extra/01-multi-tenant.md](python-fastapi-backend/extra/01-multi-tenant.md)
2. Several deployable backends — [python-fastapi-backend/extra/02-microservices.md](python-fastapi-backend/extra/02-microservices.md)
3. In-product agent / team — [python-fastapi-backend/extra/03-agent-teams.md](python-fastapi-backend/extra/03-agent-teams.md)
4. Your library / package — [python-fastapi-backend/extra/04-packages.md](python-fastapi-backend/extra/04-packages.md)
5. WebSocket / SSE — [python-fastapi-backend/extra/05-realtime.md](python-fastapi-backend/extra/05-realtime.md)
6. Outbox — [python-fastapi-backend/extra/06-outbox.md](python-fastapi-backend/extra/06-outbox.md)
7. SSO (OIDC / SAML) — [python-fastapi-backend/extra/07-sso.md](python-fastapi-backend/extra/07-sso.md)
8. Search engine — [python-fastapi-backend/extra/08-search.md](python-fastapi-backend/extra/08-search.md)
9. Customer webhooks — [python-fastapi-backend/extra/09-webhooks.md](python-fastapi-backend/extra/09-webhooks.md)
10. Hide / retain / erase — [python-fastapi-backend/extra/10-retention.md](python-fastapi-backend/extra/10-retention.md)

## Extra (Next.js)

Only if that UI **already** has the shape. Map: [nextjs-frontend/README.md](nextjs-frontend/README.md) Extra section.

1. Tenant URL — [nextjs-frontend/extra/01-multi-tenant.md](nextjs-frontend/extra/01-multi-tenant.md)
2. Several Next apps — [nextjs-frontend/extra/02-apps.md](nextjs-frontend/extra/02-apps.md)
3. i18n — [nextjs-frontend/extra/03-i18n.md](nextjs-frontend/extra/03-i18n.md)
4. SSO (browser) — [nextjs-frontend/extra/04-sso.md](nextjs-frontend/extra/04-sso.md)
5. Realtime UI — [nextjs-frontend/extra/05-realtime.md](nextjs-frontend/extra/05-realtime.md)
6. Search UI — [nextjs-frontend/extra/06-search.md](nextjs-frontend/extra/06-search.md)
7. Uploads — [nextjs-frontend/extra/07-uploads.md](nextjs-frontend/extra/07-uploads.md)
8. Agent run UI — [nextjs-frontend/extra/08-agents.md](nextjs-frontend/extra/08-agents.md)

---

## Agent operations files (coding agents only)

WHEN: you are the **IDE coding agent** — not an in-product `AgentRun` worker ([backend extra 03](python-fastapi-backend/extra/03-agent-teams.md)).
LOAD: [agents/README.md](agents/README.md) for the full WHEN → file map.

WHEN: session start → LOAD [agents/01-boundary.md](agents/01-boundary.md).
WHEN: every code turn → LOAD [agents/02-turn.md](agents/02-turn.md).
WHEN: broad or risky prompt → LOAD [agents/03-anti-patterns.md](agents/03-anti-patterns.md).
WHEN: user pasted an error → LOAD [agents/04-errors.md](agents/04-errors.md).
WHEN: unfamiliar code, new noun, or an unchecked assumption → LOAD [agents/05-understand.md](agents/05-understand.md).
WHEN: multi-file or multi-turn task → LOAD [agents/06-plan.md](agents/06-plan.md) first, every turn.
WHEN: before replying → LOAD [agents/07-verify.md](agents/07-verify.md).
WHEN: new project or empty repo → LOAD [agents/08-architecture.md](agents/08-architecture.md) before any stack file.
WHEN: product `README.md` / `docs/`, a surviving product decision, **or** a ship / handover / close → LOAD [agents/09-docs.md](agents/09-docs.md).

