# 03 · Anti-patterns (vibe coding)

WHEN: you are about to create files, load many playbook docs, or the user prompt is broad / multi-feature / long-thread continuation.
LOAD: this file and [02-turn.md](02-turn.md).
RELATED: [04-errors.md](04-errors.md) after something breaks.
SCOPE: MUST NOT behaviors. Human narrative: [for-humans 04-pitfalls](../for-humans/en/04-pitfalls.md) — do not load it; point the user there.

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

## Dependencies

WHEN: you are about to write a package name into `package.json` / `pyproject.toml`, or an `import` of something not already installed.
WHERE: the manifest at the stack root.
MUST NOT: write a package name from memory or from what a name "should" be. You do not reliably know which names exist; a name you invent is a name someone may have registered for exactly this mistake, and installing it runs its code on your machine and in CI.
HOW: confirm the package exists and is the one meant (registry page, downloads, last release, repo) **before** the manifest edit. If you cannot confirm it, say so and ask — do not install a near-match.
LOAD: [frontend 03](../nextjs-frontend/03-file-structure.md), [backend 02](../python-fastapi-backend/02-file-structure.md).

WHEN: an import fails and the fix looks like "install it."
MUST NOT: install whatever the traceback named. The name in the error is the name **you wrote** — if it was wrong, the traceback repeats your mistake back to you.
HOW: check the name first ([04-errors.md](04-errors.md) A01). A typo, a wrong casing, or a package that was renamed years ago is more likely than a missing install.

WHEN: writing a version number into a manifest.
MUST NOT: type one from memory. The version you remember is the one that was current during training — old by definition, and old is where the advisories are.
HOW: `npm install <pkg>` / `uv add <pkg>` and let it resolve. Hand-editing means reading the registry first.
MUST NOT: copy a version out of a tutorial or another project.

WHEN: reaching for a package to format a date, build a class string, debounce, deep-clone, or make a UUID.
MUST NOT: add a dependency for what the language or framework already does.

WHEN: a lockfile conflict or a resolution error blocks you.
MUST NOT: delete or regenerate the lockfile to make it go away — that silently moves every transitive version.
HOW: change the one dependency deliberately, and say in the message what moved.

WHEN: the user asks for a package upgrade or an audit finding is open.
MUST NOT: bump a framework major inside an unrelated task ([02-turn.md](02-turn.md) single slice).

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

WHEN: chat history contradicts the file on disk, the playbook, or git.
HOW: the full order of authority is in [07-verify.md](07-verify.md) — disk, then playbook, then this turn, then earlier turns.
MUST NOT: follow stale chat over the current file.

WHEN: a decision from an earlier turn matters and you are not sure what it was.
HOW: read the plan ([06-plan.md](06-plan.md)). MUST NOT: reconstruct it from the conversation — that is the thing that decays.

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

## Reaching outside the working tree

WHEN: tempted to commit, push, open a PR, write a `Dockerfile` or compose file, add a CI workflow, or run a migration against a real database.
MUST NOT: any of it unless the user asked in this conversation ([01-boundary.md](01-boundary.md) *WHAT you may run*).
HOW: name the command, say why it is needed, stop. The user runs it.

WHEN: the task feels finished and a commit "would be tidy".
MUST NOT: commit to be helpful. Finishing the edit and reporting it **is** the finished work.

WHEN: the user asked for one such action ("commit this").
HOW: do that one. MUST NOT: treat it as permission for the next one — a commit is not a push.

---

## Meta

WHEN: any turn.
MUST NOT: stop without stating what remains vs user's done-when list. Playbook-vs-application paths: [01-boundary.md](01-boundary.md).

---

## Done

MUST: confirm this turn avoided every MUST NOT above that applied.
MUST: confirm scope matches single-slice rule in [02-turn.md](02-turn.md).
