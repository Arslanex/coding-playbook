# Coding Playbook

**Human readers:** this repo is a playbook for your team and your coding agents — not runnable app code. Start with **[USER-GUIDE.md](USER-GUIDE.md)** (EN + TR). Prompts: **[HOW-TO-PROMPT.md](HOW-TO-PROMPT.md)**. Vibe coding risks: **[VIBE-CODING-PITFALLS.md](VIBE-CODING-PITFALLS.md)**. Error checklist (50): **[VIBE-CODING-ERRORS.md](VIBE-CODING-ERRORS.md)** (EN + TR).

**İnsan okuyucular:** **[USER-GUIDE.md](USER-GUIDE.md)** · **[HOW-TO-PROMPT.md](HOW-TO-PROMPT.md)** · **[VIBE-CODING-PITFALLS.md](VIBE-CODING-PITFALLS.md)** · **[VIBE-CODING-ERRORS.md](VIBE-CODING-ERRORS.md)** (50 hata, EN + TR).

---

## Agent instructions

Everything below is written for **AI coding agents** (`WHEN` / `LOAD` / `MUST`). Humans can skip to [USER-GUIDE.md](USER-GUIDE.md).

WHEN: creating or editing application code, **or** changing a playbook file because this project decided differently.
LOAD: this file first. Then follow **Agent LOAD map** below. MUST NOT: load every stack. MUST NOT: load human prose guides (`USER-GUIDE.md`, `HOW-TO-PROMPT.md`, `VIBE-CODING-*.md`) unless the user `@`-references them.

This folder is playbook, not application source. MUST NOT: copy it into `src/`.

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
HOW: point to [USER-GUIDE.md](USER-GUIDE.md) and [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md) — MUST NOT load them yourself unless `@`-referenced.

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

Human mirrors (users only): [USER-GUIDE.md](USER-GUIDE.md) · [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md) · [VIBE-CODING-PITFALLS.md](VIBE-CODING-PITFALLS.md) · [VIBE-CODING-ERRORS.md](VIBE-CODING-ERRORS.md).
