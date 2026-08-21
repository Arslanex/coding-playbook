# Start here

> **For humans** — product owners, tech leads, and developers who adopt or maintain this playbook.

The numbered files in each stack speak `WHEN` / `LOAD` / `MUST` — that syntax is aimed at **AI coding agents**. You do not need it to use this repo. Read this page, then go to the guide that matches what you are doing.

**Türkçe:** [01-start-here.md](../tr/01-start-here.md) · **Index:** [for-humans/](../README.md) · **Agents read:** [agents/README.md](../../agents/README.md)

---

## What this repo is

**Coding Playbook** is a reference library of architecture and coding rules for two stacks:

| Stack | Folder |
|-------|--------|
| Python + FastAPI (API, workers) | [`python-fastapi-backend/`](../../python-fastapi-backend/README.md) |
| Next.js (UI) | [`nextjs-frontend/`](../../nextjs-frontend/README.md) |

It is **not** application source code. Do not copy the whole folder into `src/`. Use it as a **contract** your team and your agents follow while building the real app elsewhere.

## How to use it in your project

1. **Keep it separate.** Clone or submodule this repo beside your app, or copy only the stack folder you need. The playbook stays in something like `coding-playbook/`; your app stays in `backend/`, `frontend/`, etc.

2. **Pick one stack map.** Open the README for the stack you are building:
   - Backend → [`python-fastapi-backend/README.md`](../../python-fastapi-backend/README.md)
   - Frontend → [`nextjs-frontend/README.md`](../../nextjs-frontend/README.md)

3. **Read files in order when onboarding.** Each stack has files `01`–`16` (principles → structure → config → … → tests / security / performance). You do not need every file on day one; start with `01`, `02`, and `03`, then open the file that matches what you are building (auth, forms, workers, …).

4. **Use Extra only when the feature already exists.** Folders under `extra/` (multi-tenant, SSO, agents, search, …) add rules on top of `01`–`16`. Do not scaffold those shapes just because the doc exists.

5. **Adapt rules to your product.** These files are a starting map, not immutable law. When a rule conflicts with your stack, host, or shipped design, **change the playbook file** and leave a one-line reason in that file. That way the next person (or agent) reads the decision in git, not in chat history.

6. **Point agents at the playbook.** In Cursor (or similar), add a rule or `@` reference: load [`README.md`](../../README.md) first, then the stack README, then only the numbered file for the current task. Agents should not load all sixteen files or every Extra topic at once. For prompt templates, config pitfalls, and long-chat fixes: **[02 How to prompt](02-how-to-prompt.md)**.

## Product documentation

This playbook is rules. The **application repo** still needs its own README and `docs/` — that is what the next person (and the next agent) reads about *your* product.

Day one, after the architecture questions: `docs/data-model.md` and `docs/architecture/overview.md`. The rest of the tree is created when that shape exists, not as empty stubs. Agents follow [`agents/09-docs.md`](../../agents/09-docs.md). Task notes live in `.agent/plan.md` (git-ignored), not in chat.

```
repo/
├── README.md                          # index, not the technical document
├── CHANGELOG.md                       # what shipped, for users
└── docs/
    ├── data-model.md                  # nouns → grows ER / validation
    ├── api.md                         # when a public HTTP contract exists
    ├── security.md                    # when auth or personal data exists
    ├── known-issues.md                # first ship or handover — what is unfinished
    ├── handover.md                    # when another team takes the product
    ├── lessons-learned.md             # when a project or phase closes
    ├── architecture/
    │   ├── overview.md                # how the system works *now*
    │   └── decisions/                 # ADRs: why, not what
    ├── development/
    │   ├── setup.md
    │   ├── development-guide.md
    │   └── testing.md
    └── operations/
        ├── deployment.md              # how we publish, and publish again
        ├── runbook.md                 # how production is operated
        └── slo.md                     # agreed targets, when they exist
```

Do not put secrets in any of these files. If the code's architecture or API moves, the matching doc moves in the same change. At ship or handover: update the living architecture / data-model / API — do not open a "final architecture" file. Known issues exist so a delivery cannot look complete when it is not.

## Typical workflows

| Goal | Where to start |
|------|----------------|
| New FastAPI service | `python-fastapi-backend/01` → `02` → `03`, then topic files |
| New Next.js app | `nextjs-frontend/01` (design) → `02` → `03`, then topic files |
| Code review checklist | `15-security.md` and `14-testing.md` for that stack |
| Add SSO / tenants / agents later | Matching file under `extra/` **after** the feature exists |
| Ship, handover, or close a project | [`agents/09-docs.md`](../../agents/09-docs.md) — changelog, known issues, runbook; living docs, not a new architecture file |

## What to ignore as a human reader

- `WHEN:` / `LOAD:` / `MUST NOT:` lines are routing instructions for agents. Skim them or read the prose sections underneath.
- Content below the line **“Agent instructions”** in [`README.md`](../../README.md) is the same agent-oriented index; use this user guide instead for onboarding.

---

**Next:** [01 Start here](01-start-here.md) · [02 How to prompt](02-how-to-prompt.md) · [03 Review agent code](03-review-agent-code.md) · [04 Pitfalls](04-pitfalls.md) · [05 Errors](05-errors.md) · [06 Glossary](06-glossary.md)
