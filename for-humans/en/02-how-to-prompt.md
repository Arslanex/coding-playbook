# How to prompt coding agents

> **For humans** — anyone writing prompts to Cursor, Copilot, Claude Code, or a similar tool.

This is not a rule file. It teaches **you** how to phrase a task so the agent stays small, correct, and fast. The agent's own version of this is [agents/02-turn.md](../../agents/02-turn.md) — do not send it this page.

**Türkçe:** [02-how-to-prompt.md](../tr/02-how-to-prompt.md) · **Index:** [for-humans/](../README.md) · **Agents read:** [agents/README.md](../../agents/README.md)

---

## One sentence rule

**Say what to change, where it lives, which playbook files apply, and what “done” looks like — then stop.**

Do not paste the whole playbook into chat. Do not ask for “everything” in one prompt.

---

## Cursor / IDE setup (do this once)

Put the playbook where the agent can `@`-reference it (submodule, sibling folder, or monorepo path).

**Project rule (recommended)** — add to `.cursor/rules` or your IDE’s project instructions:

```text
Application code: follow coding-playbook/README.md routing.
Load only the stack README + the numbered file for this task (+ that file's LOAD: siblings).
Do not load all 01–16 or every extra/ topic.
If a playbook rule conflicts with this repo, adapt the playbook file and commit the reason — do not invent a parallel layout in src/.
Minimize scope: one task per agent turn when possible.
```

**Per chat:** `@coding-playbook/README.md` plus the stack README and **one** topic file (e.g. `@python-fastapi-backend/03-config.md`). Attach the **application** files being edited, not the entire repo.

| Setup | Good | Bad |
|-------|------|-----|
| Rules | Short pointer to playbook + “load minimal files” | Pasting all of `01`–`16` into rules |
| Context | Task file + 1–3 playbook files | `@` entire `coding-playbook/` folder |
| Chat length | New chat per feature slice | One chat for “build backend + frontend + deploy” |

---

## Prompt shape (copy and fill)

```text
Stack: [python-fastapi-backend | nextjs-frontend]
Task: [one sentence — what changes]
Files: [paths in the APPLICATION repo you are editing]
Playbook: load [stack README] + [numbered file, e.g. 03-config]
Constraints:
  - [optional: must not touch X, keep diff small, no new deps]
Done when:
  - [observable checklist — tests pass, env in Settings/env.ts, etc.]
```

**Example — backend config**

```text
Stack: python-fastapi-backend
Task: Add REDIS_URL and redis socket timeout to Settings; wire into infra/cache client.
Files: backend/src/config/settings.py, backend/src/infra/cache/redis.py
Playbook: load python-fastapi-backend/README.md + 03-config.md (+ 08-infra if you add a new infra folder)
Constraints:
  - No os.getenv outside config/
  - No magic numbers — timeout is a Settings field
Done when:
  - .env.example lists REDIS_URL and REDIS_SOCKET_TIMEOUT
  - get_settings() used in redis client
  - Existing tests still pass
```

**Example — frontend env**

```text
Stack: nextjs-frontend
Task: Add server-side API_BASE_URL to lib/env.ts and use it in the fetch wrapper.
Files: frontend/lib/env.ts, frontend/lib/api/client.ts
Playbook: load nextjs-frontend/README.md + 04-config.md + 09-api-client.md
Constraints:
  - No process.env outside lib/env.ts
  - No NEXT_PUBLIC_ for secrets
Done when:
  - Zod parse for API_BASE_URL
  - client.ts imports serverEnv only from lib/env.ts
```

---

## While writing config (most repeated mistakes)

Config prompts fail when the task is vague. Be explicit:

| Say this | Not this |
|----------|----------|
| “Add field `X` to Settings / `lib/env.ts`” | “Set up config” |
| “Operational limit only; product rule stays in modules/” | “Add constants file” |
| “Update `.env.example` names only” | “Fix env” |
| “Backend: pydantic Settings frozen=True” | “Use env vars” |
| “Frontend: server vs NEXT_PUBLIC_ — browser must not see Y” | “Add API key to frontend” |

Backend playbook: [python-fastapi-backend/03-config.md](../../python-fastapi-backend/03-config.md)  
Frontend playbook: [nextjs-frontend/04-config.md](../../nextjs-frontend/04-config.md)

**Split backend vs frontend config in separate prompts.** One agent turn touching both stacks doubles load errors and repeated explanations.

---

## Prompts by task type

| You want | Playbook files to `@` | Prompt tip |
|----------|----------------------|------------|
| New API endpoint | `09-modules`, `10-http`, `12-api` | Name the module noun; say “router mounts in http/router.py only” |
| DB column + migration | `06-database`, `07-migrations` | Model change and Alembic revision in **same** prompt |
| Background job | `11-workers`, relevant module | “Never BackgroundTasks; queue + worker job file” |
| New screen | `01-design`, `03-file-structure`, `10-features` | UI path under `features/<noun>/`, not `app/` city |
| Auth / cookies | backend `13-identity-security`, frontend `12-auth` | Two prompts if both stacks; cite cookie/JWT split |
| PR / security pass | `15-security` (+ `14-testing`) | “Review diff against checklist; do not refactor unrelated” |

Open a file’s `RELATED:` line **only** if your task is also that topic — do not preload siblings “just in case.”

---

## Biggest pain points (and fixes)

### 1. Repeating the playbook in every message

**Symptom:** You re-paste MUST/MUST NOT blocks; the agent still drifts.  
**Fix:** One `@README.md` + one numbered file per turn. Put standing rules in **project rules**, not chat. Say: “Follow loaded playbook files; do not restate them in your reply.”

### 2. Context bloat near the end of a long chat

**Symptom:** Early turns were good; later turns ignore constraints, duplicate files, or “fix” working code.  
**Fix:** **New chat** per slice (one module, one migration, one screen). Re-attach only: task description + 2–3 playbook files + current app files. Summarize prior decision in **three lines**, not full thread history.

### 3. Loading everything

**Symptom:** Agent reads all 16 files or all of `extra/`; slow, contradictory, over-scaffolded.  
**Fix:** Prompt must name **one** numbered file. Explicit: “Do not load other numbered files unless this file’s LOAD: line requires them.”

### 4. Extra topics too early

**Symptom:** `src/agents/`, `extra/i18n`, outbox tables appear before the product needs them.  
**Fix:** “Do not load or scaffold extra/ unless [feature] already exists in this repo.” Extra is for **shipped** shapes only.

### 5. Parallel layout in `src/`

**Symptom:** `utils/`, `helpers/`, `services/` next to `modules/`, or FastAPI `http/modules/infra` copied into Next.js.  
**Fix:** “Place files per playbook 02/03 file-structure. If our repo differs, update the playbook file and say why — do not silently invent a second tree.”

### 6. Config sprawl

**Symptom:** `os.getenv` in modules, magic numbers in repositories, secrets on `NEXT_PUBLIC_`.  
**Fix:** Dedicated config prompt (see examples). Done criteria must include “no env reads outside config/ or lib/env.ts”.

### 7. “Build the whole feature” in one prompt

**Symptom:** Huge diff, missed tests, wrong layer boundaries.  
**Fix:** Chain prompts: (1) model + migration → (2) service + API → (3) worker if needed → (4) UI feature package → (5) tests. Each step names **done when**.

### 8. Playbook vs chat disagreement

**Symptom:** Team decided X in Slack; playbook still says Y; next agent reverts X.  
**Fix:** Prompt: “We adapted rule Z — update `python-fastapi-backend/NN-….md` SCOPE with one-line reason, then implement.” Decisions live in git, not chat.

### 9. Mixing coding agents with in-product LLM agents

**Symptom:** Confusion between Cursor editing code vs `AgentRun` / transcript UI in the product.  
**Fix:** Coding agents use `01`–`16`. In-product agents use [extra/03-agent-teams](../../python-fastapi-backend/extra/03-agent-teams.md) only when **your product** runs LLM jobs — say which you mean in the prompt.

### 10. No verifiable “done”

**Symptom:** Agent stops mid-task or argues about completeness.  
**Fix:** Always end with a **Done when** checklist (tests, files touched, env names, no new folders).

---

## Anti-patterns (short)

- “Follow best practices” without stack or file paths  
- “Read all playbook files first”  
- “Refactor while you’re here” without scope  
- Pasting 200-line diffs back into chat for “continue”  
- Asking the agent to copy `coding-playbook/` into `src/`  
- Re-explaining architecture the numbered file already states  

---

## Good vs weak prompt (same task)

**Weak**

```text
Add Redis to the project and configure everything properly.
```

**Strong**

```text
Stack: python-fastapi-backend
Task: Add Redis client in infra/cache; settings fields REDIS_URL and REDIS_SOCKET_TIMEOUT.
Files: backend/src/config/settings.py, backend/src/infra/cache/redis.py, backend/.env.example
Playbook: README.md + python-fastapi-backend/03-config.md + 08-infra.md
Done when: no os.getenv outside config/; .env.example updated; unit test mocks redis get/set
Do not: add workers, caching policy, or frontend changes in this turn.
```

---

**Next:** [01 Start here](01-start-here.md) · [02 How to prompt](02-how-to-prompt.md) · [03 Review agent code](03-review-agent-code.md) · [04 Pitfalls](04-pitfalls.md) · [05 Errors](05-errors.md) · [06 Glossary](06-glossary.md)
