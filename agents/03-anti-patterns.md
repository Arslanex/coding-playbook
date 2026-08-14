# 03 · Anti-patterns (vibe coding)

WHEN: you are about to create files, load many playbook docs, or the user prompt is broad / multi-feature / long-thread continuation.
LOAD: this file and [02-turn.md](02-turn.md).
RELATED: [04-errors.md](04-errors.md) after something breaks.
SCOPE: MUST NOT behaviors. Human narrative: [VIBE-CODING-PITFALLS.md](../VIBE-CODING-PITFALLS.md).

---

## Prompt and scope

WHEN: user message is only a category word ("dashboard", "CRM", "auth", "fix it").
MUST NOT: infer full product requirements from that word alone.
HOW: ask for stack, paths, data model, or verbatim error ([02-turn.md](02-turn.md)).

WHEN: starting a turn.
MUST NOT: load all sixteen numbered files or all of `extra/`.
MUST NOT: implement backend + frontend + deploy + infra in one diff unless user explicitly ordered that.

---

## Layout and architecture

WHEN: tempted to add a catch-all folder or duplicate service layer.
WHERE: application `src/` or `frontend/`.
MUST NOT: add `utils/`, `helpers/`, `common.py`, or a second `*Service` in the same package.
LOAD if unsure: [backend 01](../python-fastapi-backend/01-coding-principles.md).

WHEN: adding UI or HTTP code.
WHERE: frontend uses `features/` and `app/` shell; backend uses `modules/` and `http/` shell.
MUST NOT: put business rules in `http/`.
MUST NOT: put product UI as a city under root `app/`.
LOAD: [frontend 03](../nextjs-frontend/03-file-structure.md).

WHEN: working on frontend.
MUST NOT: copy backend folder names (`http/`, `modules/`, `infra/`) into `frontend/`.
MUST NOT: connect Next.js to Postgres.
MUST NOT: store JWT in `localStorage`.
LOAD: [frontend README](../nextjs-frontend/README.md).

WHEN: product tree differs from playbook.
WHERE: playbook numbered file in `coding-playbook/`.
HOW: update playbook with one-line reason — MUST NOT silently fork layout in `src/`.

---

## Extra and in-product agents

WHEN: Extra doc exists but product repo has no matching feature yet.
MUST NOT: scaffold multi-tenant, SSO, outbox, `modules/agents/`, i18n routes, etc.

WHEN: task is you editing code in the IDE.
HOW: use [agents/](README.md) and stack `01`–`16`.

WHEN: task is in-product LLM runs (`AgentRun`, transcript, worker agent jobs).
LOAD: [backend extra 03](../python-fastapi-backend/extra/03-agent-teams.md), [frontend extra 08](../nextjs-frontend/extra/08-agents.md) — only when that runtime code is in scope.

---

## Config and secrets

WHEN: reading or adding env vars, limits, timeouts, pool sizes.
WHERE (backend): `src/config/` only for raw env read.
WHERE (frontend): `lib/env.ts` only for `process.env`.
MUST NOT: `os.getenv` or `process.env` in features, modules, infra, workers, or components.
MUST NOT: magic numbers for pools/timeouts — they belong in `Settings` / `lib/env.ts`.
MUST NOT: API keys or session secrets on `NEXT_PUBLIC_*`.
LOAD: [backend 03](../python-fastapi-backend/03-config.md), [frontend 04](../nextjs-frontend/04-config.md).

---

## Security and review

WHEN: shipping auth, uploads, webhooks, or new routes.
LOAD: stack `15-security` before merge.

WHEN: UI hides a button for unauthorized users.
MUST NOT: skip permission check in backend service.
LOAD: [backend 13](../python-fastapi-backend/13-identity-security.md).

WHEN: building SQL from user input.
MUST NOT: string-concatenate into queries.
LOAD: [backend 06](../python-fastapi-backend/06-database.md).

WHEN: you finished generated code.
HOW: treat output as draft — user still reviews; you still follow security files.

---

## Happy path only

WHEN: adding data-fetching UI.
WHERE: the feature component.
MUST NOT: ship success path only — need loading, empty, error, success.
LOAD: [frontend 01](../nextjs-frontend/01-design.md).

WHEN: fetching read data on Next.js.
MUST NOT: default to `useEffect` GET when server component fetch works.
LOAD: [frontend 07](../nextjs-frontend/07-data.md).

WHEN: work takes longer than HTTP timeout or must survive crash.
WHERE: worker queue and job file — not route handler, not `BackgroundTasks`.
LOAD: [backend 11](../python-fastapi-backend/11-workers.md).

---

## Context and chat history

WHEN: chat history contradicts loaded playbook or git.
HOW: playbook + git win — MUST NOT follow stale chat over rules.

WHEN: user pasted a specific error.
HOW: minimal fix for that error only ([04-errors.md](04-errors.md)).
MUST NOT: "fix" working code unrelated to the error.

WHEN: prior turns clearly drifted (random regressions, forgotten constraints).
HOW: tell user to start fresh chat with task + 2–3 playbook files + current app paths — you cannot start chats yourself.

---

## Playbook vs chat

WHEN: user states an adapted rule that differs from playbook text.
WHERE: matching playbook file.
HOW: update playbook with reason, then code to match.
MUST NOT: revert git/playbook decisions because old chat said otherwise.

---

## Meta

WHEN: any turn.
MUST NOT: copy `coding-playbook/` into application `src/`.
MUST NOT: stop without stating what remains vs user's done-when list.

---

## Done

MUST: confirm this turn avoided every MUST NOT above that applied.
MUST: confirm scope matches single-slice rule in [02-turn.md](02-turn.md).
