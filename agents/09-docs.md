# 09 · Product docs, and where notes go

WHEN: you are creating or updating `README.md` or `docs/` in the **product** repo; a code change that moves architecture, the data model, a public API, authz, how the app is run, or how it is deployed; a decision that must survive this task; you are about to write a note so a later turn "remembers"; **or** a version is shipping, production is being handed over, or a project/phase is closing.
LOAD: this file. The two day-one documents: [08-architecture.md](08-architecture.md). The task plan (not a product doc): [06-plan.md](06-plan.md).
RELATED: [05-understand.md](05-understand.md) — read the matching product doc before assuming · [07-verify.md](07-verify.md) — a doc that no longer matches the code is an unmet slice.
SCOPE: where writing goes, and the committed docs tree in the product repo. Reason this file exists: 08 only names the two day-one documents; the rest of `docs/` and the difference between a task note and a product doc had no home. Ship/handover files were added so a delivery does not look finished when debt and runbooks are missing. ADAPTED (GİRVAK): day-one data model lives at `docs/architecture/data-model.md` beside `overview.md`, not at `docs/data-model.md`. HTTP contract lives at `docs/architecture/api.md` beside overview and data-model. Version roadmap lives at `docs/plan/backend/` and `docs/plan/frontend/` (`v1.md`, `v2.md`, … with checkbox slices) — not in `.agent/plan.md`. Product art direction lives at `docs/design/` (`README.md` + one or more named system files, e.g. `swiss-editorial.md`) — not in the playbook and not in `docs/architecture/`.

Your context is not memory. Chat is not a filing cabinet. Anything that must outlive this turn is a file, and the file's **kind** decides whether it is ignored, committed as product truth, or written into this playbook.

---

## Three places writing goes

Stop at the first match. There is no fourth place.

**1. This task** — `.agent/plan.md` in the product repo, git-ignored.
WHERE: [06-plan.md](06-plan.md). Slices, `Done when`, `## Open`, `## Decisions` that only this task needs.
MUST NOT: the plan in the chat. MUST NOT: the plan inside `coding-playbook/`.

**2. This product** — `README.md` and `docs/` in the product repo, committed.
WHERE: the tree below. What the system is, why a choice was made, how a new person runs it, what the API and data look like, what security/privacy this product actually does.
MUST: a human can review it. MUST: it stays true when the code that it describes changes — same change set, not a follow-up you will forget.

**3. This playbook** — the matching numbered or Extra file in `coding-playbook/`.
WHEN: a rule of *how we write code* changed for this product ([AGENTS.md](../AGENTS.md), *Not carved in stone*).
MUST NOT: put a playbook adaptation only in `docs/`. The next coding agent loads the playbook file, not a paragraph in the product README.

MUST NOT: a `.agent/notes.md`, a `NOTES.md`, a Cursor memory, or a chat recap as the place a decision lives. That is the conversation with extra steps, and the next agent will not load it.
MUST NOT: copy this playbook into `docs/`. Point at `coding-playbook/` from the README; do not fork the rules.

WHEN: you learned something mid-turn.
HOW: write it now, in the matching place above — not at the end of the task.

| It must survive… | It goes in |
|---|---|
| the next slice of this task | the plan (`## Decisions` / `## Open`) |
| the next agent on this product | `docs/` or an ADR |
| the next agent on every task | the playbook file |

---

## Tree (product repo)

```
repo/
├── README.md
├── CHANGELOG.md                       # when a version ships to users
├── docs/
│   ├── security.md                    # when auth or personal data exists
│   ├── known-issues.md                # first ship or handover — what is unfinished
│   ├── handover.md                    # when leaving the product to another team
│   ├── lessons-learned.md             # when a project or phase closes
│   ├── architecture/
│   │   ├── overview.md
│   │   ├── data-model.md              # nouns → grows ER / validation
│   │   ├── api.md                     # when a public HTTP contract exists
│   │   └── decisions/
│   │       ├── ADR-001-short-kebab.md
│   │       └── ADR-002-short-kebab.md
│   ├── plan/
│   │   ├── README.md                  # index — version roadmap, not task scaffolding
│   │   ├── backend/
│   │   │   ├── v1.md                  # checkbox slices for backend v1
│   │   │   └── v2.md                  # backend backlog
│   │   └── frontend/
│   │       ├── v1.md                  # checkbox slices for frontend v1
│   │       └── v2.md                  # frontend backlog
│   ├── design/
│   │   ├── README.md                  # index — when to load; link to system doc(s)
│   │   └── <system-name>.md           # product art direction (e.g. swiss-editorial.md)
│   ├── development/
│   │   ├── setup.md
│   │   ├── development-guide.md
│   │   └── testing.md
│   └── operations/
│       ├── deployment.md              # when the product approaches production
│       ├── runbook.md                 # when production is operated
│       └── slo.md                     # when SLOs are agreed
└── …
```

MUST: paths are in the **product** repo, not inside `coding-playbook/`.
MUST NOT: a parallel tree (`documentation/`, `wiki/`, `adr/` at repo root) while this file names `docs/`.
WHEN: the product already shipped a different docs layout.
HOW: adapt this file with a one-line reason ([AGENTS.md](../AGENTS.md), *Not carved in stone*) — MUST NOT: silently write the old paths.

OpenAPI YAML/JSON, if it is a file, lives next to the code that serves it (backend). `docs/architecture/api.md` says **where**, and the generate command if it is produced from annotations. MUST NOT: a hand-copied spec that drifts from the running app.

---

## When each file exists

MUST NOT: scaffold the tree with empty heading stubs on day one. A file with only headings is fiction, and the next agent will believe it. Same gate as Extra: the file exists when the shape exists.

| File | Create when | Not before |
|---|---|---|
| `README.md` | first commit of the product | — |
| `docs/architecture/data-model.md` | 08's six questions are answered | any migration |
| `docs/architecture/overview.md` | 08's six questions are answered | any module |
| `docs/architecture/decisions/ADR-*.md` | a choice a later agent would re-open | "we use X" as a fact already in overview |
| `docs/development/setup.md` | the skeleton can be run locally (08 step 0) | secrets exist to paste |
| `docs/development/development-guide.md` | the first PR, or a convention this product has that the playbook does not | copying the playbook into prose |
| `docs/development/testing.md` | tests exist | inventing a coverage number |
| `docs/architecture/api.md` | the first public URL exists | the data model is still only nouns |
| `docs/plan/README.md` + `docs/plan/{backend,frontend}/v*.md` | the first version roadmap is agreed (usually with 08) | empty checkbox stubs before scope exists |
| `docs/design/README.md` + `docs/design/<system-name>.md` | frontend UI work is in scope and art direction is agreed | empty typography/grid stubs before a public site exists |
| `docs/security.md` | identity exists, or a column holds personal data | an empty RBAC matrix |
| `docs/operations/deployment.md` | a real environment besides local, or a release job | a `Dockerfile` nobody asked for |
| `CHANGELOG.md` | a version ships to users | a feature list copied into the deploy doc |
| `docs/operations/runbook.md` | someone other than the authors will operate production | inventing alerts the product does not emit |
| `docs/operations/slo.md` | the team agreed latency / error / availability numbers | a `99.9%` you made up |
| `docs/known-issues.md` | first production ship **or** handover, whichever is first | claiming the delivery is complete while debt is only in chat |
| `docs/handover.md` | another team will own the product | pasting runbook + architecture into a new file |
| `docs/lessons-learned.md` | a project or a named phase closes | a blame list or a secret from a postmortem |

WHEN: 08 is running (new product, no code yet).
HOW: write **only** `docs/architecture/data-model.md` and `docs/architecture/overview.md`, and the user has seen them. README if the repo has none — short, as the index below. Nothing else in this tree until the row above matches.

WHEN: a version is shipping, or the product is handed over.
HOW: the ship files below. MUST: living architecture / data-model / API describe **the running system**, not the first design — reconcile, do not open a `final-architecture.md`.

---

## README.md

First file a new developer opens. They should understand the project in 5–10 minutes.

MUST be an **index**, not the technical document. Detail lives in `docs/`. The README links there.

Headings:

- What this project is
- What it is for
- Architecture in a few lines (link `docs/architecture/overview.md`)
- Stack / technologies
- Requirements
- Local setup (link `docs/development/setup.md` once that file exists)
- How to run
- How to run tests (link `docs/development/testing.md` once that file exists)
- Design / UI system (link `docs/design/README.md` once frontend art direction exists)
- Repository layout
- Links to the docs that exist **today**
- Ownership / who to ask
- Changelog (link `CHANGELOG.md` once it exists)

MUST NOT: paste the data model, the endpoint list, or the ADR history into the README.
MUST NOT: let the README rot while `docs/` is current — the README's job is to point.

---

## docs/architecture/overview.md

Question: how does this system actually work?

Day one this is 08's short document: actors and roles, isolation, who owns writes and authz (FastAPI, always), outside systems in use **today**, what is out of scope. Ten to thirty lines. It **grows** as components exist — it does not start as a diagram of a system nobody asked for.

Headings, filled only when the thing exists:

- System overview
- Architecture diagram (or a few lines that would be the diagram)
- Components / services and what each owns
- Data flow
- External integrations
- Authentication / authorization (the model; the matrix lives in `docs/security.md`)
- Infrastructure in use **today**
- Technical principles this product actually follows
- Known technical limits
- Out of scope

WHEN: the code's shape moved (a new component, a new integration, isolation changed).
HOW: edit this file in the **same change set** as the code. Name the doc change in the reply ([08-architecture.md](08-architecture.md) *When the agreement changes*).
MUST NOT: expand it with what the product might need later.
MUST NOT: a second architecture file (`docs/architecture.md` at the `docs/` root, `ARCHITECTURE.md` at repo root).

---

## docs/architecture/decisions/ (ADRs)

Question: **why** did we decide this? Not "what do we use" — overview already says that.

WHEN: you decide anything a later turn or a later agent could reasonably re-open — a database, a cache, an auth scheme, "we are not doing X".
HOW: one file `docs/architecture/decisions/ADR-NNN-short-kebab.md`. Take the next number. Write it **when the decision is made**, not at the end of the task.

Each ADR contains:

- Context / Problem
- Alternatives
- Decision
- Why
- Consequences
- Date
- Status — `accepted` · `superseded` (link the successor) · `rejected`

The plan's `## Decisions` line is the seed. If the decision must survive the task, the ADR is the record; delete nothing from the plan except to point at the ADR.

MUST NOT: an ADR that only restates overview ("we use PostgreSQL") with no alternative and no why.
MUST NOT: a decision that contradicts a playbook rule living **only** here — that also goes in the playbook file ([AGENTS.md](../AGENTS.md)).
MUST NOT: secrets, credentials, or internal URLs that are not already public.

WHEN: the decision is reversed.
HOW: set Status to `superseded`, link `ADR-NNN` of the new choice. MUST NOT: delete the old file. The point of an ADR is the history.

---

## docs/development/setup.md

Question: how do I run this on my machine?

Headings:

- Requirements
- Versions that must match (language, runtime, toolchain)
- Environment setup
- Environment **names** (link `.env.example`; values stay out)
- Dependency install
- Database setup
- Seed / test data
- How to run the app
- Common setup failures

MUST NOT [critical]: secret values, tokens, private keys, production DSNs, or real personal data in this file or in any other file in `docs/`.
MUST: `.env.example` holds names (and non-secret defaults). This file says which names are required, not what they are set to.

---

## docs/development/development-guide.md

Question: how do we develop **in this repo**?

Headings:

- Branch strategy
- Commit convention
- PR process
- Code review rules
- Coding conventions that are **this product's**, not a paste of the playbook
- Folder structure (one short tree; detail is the stack `02` / `03` file)
- Error handling
- Logging
- Configuration
- Adding a dependency (confirm the name exists — [03-anti-patterns.md](03-anti-patterns.md))
- Migrations
- How a feature is supposed to land

MUST: link `coding-playbook/` for stack rules. MUST NOT: rewrite `01`–`16` here. This file is the team's process plus the few product-specific conventions.

---

## docs/development/testing.md

Question: how do we test this, and what is required before merge?

Headings:

- Test strategy
- Unit / integration / E2E — what each layer is for **here**
- Naming
- Test data
- Mocking
- How to run the tests
- What CI runs
- Minimum coverage **if this product has one** — MUST NOT: invent a percentage

Stack detail: backend / frontend `14-testing.md`. This file says what this repo actually runs.

---

## docs/operations/deployment.md

Question: how did we publish, and how do we publish again? Audience: developer / DevOps.

WHEN: production (or a shared staging) is in scope — not on an empty repo ([01-boundary.md](01-boundary.md) *WHAT you may run*).

This **is** the deployment and release document. MUST NOT: a second `docs/release.md` that repeats it.

Headings:

- Environments
- Build
- Production deploy steps (the repeatable path)
- Configuration per environment (names, not secret values)
- Database migration (expand/contract when the change needs it — backend `07`)
- Deploy order
- Rollback
- Release process
- CI/CD
- Checks after deploy

MUST NOT: write a `Dockerfile`, compose file, or CI workflow because this doc has a heading. The heading is filled when those exist, and they exist when the user asked ([01-boundary.md](01-boundary.md)).
MUST NOT: user-facing feature lists here — those belong in `CHANGELOG.md`.

---

## docs/architecture/data-model.md and docs/architecture/api.md

Living data model and API contract. Architecture folder holds overview, data model, and HTTP contract so day one stays small; they link to each other.

### architecture/data-model.md (day one — 08)

Enough that a migration can be written from it: nouns, fields, relationships, who owns each row.

Grows as the shape exists:

- ER diagram (or a textual schema that is as precise)
- Data-layer types (repository / DAO — what exists in `infra/db`, not a second taxonomy)
- Model validation rules the product actually enforces

MUST NOT: a column the product has no use for yet ([08-architecture.md](08-architecture.md)).
WHEN: a noun appears that this file does not have.
HOW: this file first, then the model, then the migration — same change set.

### architecture/api.md (first public URL)

Headings:

- Where the OpenAPI / Swagger spec lives (`yaml`/`json` path, or `/docs` in non-production)
- Command that **generates** it, if it is generated from annotations — MUST NOT: a second hand-maintained spec
- Main endpoints and what each is for
- Request / response examples (the JSON the client sees — backend `12`)
- API versioning (`/v1` in the path; how `/v2` is introduced — backend `12`)

MUST: examples match the running schemas. A sample payload that the schema rejects is a bug in this file.
MUST NOT: duplicate every field in prose that OpenAPI already lists. This file orients; the spec is the contract.

---

## docs/plan/

Question: what ships in each version, per stack?

Committed version roadmap — not task scaffolding (that stays in `.agent/plan.md`, [06-plan.md](06-plan.md)).

```
docs/plan/
├── README.md              # index
├── backend/
│   ├── v1.md              # checkbox slices — backend v1
│   └── v2.md              # backlog — not in v1 slices
└── frontend/
    ├── v1.md
    └── v2.md
```

MUST: each `vN.md` uses checkbox lists (`- [ ]` / `- [x]`) for slices and a short *Done when* for the release.
MUST: check an item only when evidence exists (tests, manual check, docs match code) — same bar as [07-verify.md](07-verify.md).
MUST: update the matching `vN.md` when a version slice completes — same change set as the code.
MUST NOT: duplicate the full task plan from `.agent/plan.md`; the plan is ephemeral, `docs/plan/` is product truth.

WHEN: a new major version is scoped.
HOW: add `vN.md` under the stack folder; move backlog items from the previous `v(N+1).md` or `v2.md` as needed.

---

## docs/design/

Question: what does **this product's public UI** look and feel like?

Product visual and interaction system for the frontend — grid, typography, color, spacing, components, page composition, anti-patterns. **Not** backend architecture. **Not** a paste of the stack playbook.

```
docs/design/
├── README.md                  # index — when to load; links to system doc(s)
└── <system-name>.md           # one art-direction file per agreed system (kebab-case)
```

Example: `docs/design/swiss-editorial.md` for a Swiss / International Typographic Style bulletin site.

### Playbook vs product design

| Layer | Where | What it covers |
|---|---|---|
| Stack playbook | `nextjs-frontend/01-design.md` (or the frontend stack's `01-design`) | Five UI questions, loading/empty/error states, anti-slop, accessibility baseline |
| Product | `docs/design/<system-name>.md` | Art direction for **this** site — type scale, grid, palette, component character, editorial tone |

MUST: load the stack `01-design` **and** the matching product design doc before layout, CSS tokens, or page composition.
MUST: on composition, typography, and visual character, **the product doc wins** over generic SaaS/blog templates in the playbook.
MUST NOT: put product art direction only in `docs/plan/frontend/v*.md` — the plan is slices; design is the living system spec.
MUST NOT: duplicate the playbook's four UI states in prose here — link the stack file; this folder is what makes the product visually distinct.

### README.md (index)

Short index only:

- What the design system is for (which site / surface)
- Table linking each `<system-name>.md` and its purpose
- **When:** any frontend UI work (layout shell, CSS tokens, page composition)
- One line on playbook relationship (stack `01-design` for states; this folder for art direction)

MUST NOT: paste the full system into README — detail lives in the named file.

### `<system-name>.md` (art direction)

Create when frontend UI is in scope and the team has agreed a direction — usually before the first layout/CSS slice, not on day one of a backend-only repo.

Typical sections (filled when true, not as empty stubs):

- Scope — which app / routes / locales
- Product mapping — design terms → product nouns (categories, locales, platform tone)
- Core philosophy — editorial character, what to avoid
- Grid and breakpoints
- Typography — family, scale, weights
- Color and contrast
- Spacing and rhythm
- Components — nav, cards, article layout, metadata, forms (if any)
- Page patterns — home, list, detail, 404
- Anti-patterns — explicit rejects (e.g. SaaS dashboard, gradient hero)
- Relationship to API — which public fields drive which UI (link `docs/architecture/api.md`)

MUST: name real product mappings (locales, category slugs, domain) where they affect layout or copy.
MUST: update when shipped UI diverges — same change set as the frontend code ([*Same change set*](#same-change-set)).
MUST NOT: secrets, production URLs with credentials, or customer content as "examples".

WHEN: a second visual system is agreed (e.g. admin panel vs public site).
HOW: add another `<system-name>.md` and link it from `README.md`. MUST NOT: one file that mixes two unrelated surfaces without a heading that says which is which.

---

## docs/security.md

Question: what does **this product** do with secrets, personal data, and access?

WHEN: identity exists, or a column holds personal data. Playbook controls stay in stack `15` (and Extra `10` when hide/retain/erase is shipped). This file is the product's actual periods, matrix, and tools — not a paste of `15`.

Headings, filled only when true:

- Encryption — at rest (database/volume) and in transit (TLS version the platform actually uses)
- Retention periods (KVKK / GDPR) per class of data
- Erasure procedure (right to erasure; hard delete vs soft delete vs anonymise — Extra `10` if that shape ships)
- Masking / anonymisation rules
- Audit log: what is recorded, format, how long it is kept (no PII in the log body — backend `04`, `15`)
- RBAC matrix: which role may read or write which noun (roles from 08 question B; empty matrix = do not create this heading)
- Security tests (SAST / DAST) and how often, **if this product runs them**
- What is sent to third parties, and the bound on that data

MUST NOT [critical]: production secrets, real personal data, or a dump of another user's records as an "example".
MUST NOT: a security.md that says "we encrypt" with no where and no what. A vague control is not a control.

---

## CHANGELOG.md

Question: what shipped, in language a user / customer / product person can use?

WHEN: a version goes to users. One entry per release, newest first.

Headings per entry:

- Version and date
- New features
- Changes
- Bug fixes
- Breaking changes
- Known problems (short; detail in `docs/known-issues.md`)
- Changes that affect the user

MUST: breaking changes are explicit. MUST NOT: only a git log. MUST NOT: internal refactors that no user can see, unless they change behaviour.
WHERE: product repo root (`CHANGELOG.md`). WHEN: the product already uses `docs/releases.md` instead — adapt this file's path with a one-line reason; MUST NOT: keep both in conflict.

---

## docs/operations/runbook.md

Question: how is production **operated**? Audience: the on-call / operations team.

WHEN: someone other than the authors will run production. Deploy steps stay in `deployment.md`; this file is what to do **after** it is up, and when it is not.

Headings:

- Monitoring — what is watched, where the dashboard is
- Alerts — what fires, who gets it (thresholds that are product policy live in `slo.md` and are linked)
- Where logs are
- Common problems
- Steps when it is broken
- Restart / recovery
- Backup / restore
- Critical services
- Dependencies (the systems this one needs **today**)
- Who to ask

MUST NOT [critical]: production passwords, private keys, or a paste of `.env`. Access is named (which vault, which role), not valued.
MUST NOT: copy `deployment.md` into this file. Link it.

---

## docs/operations/slo.md

Question: what operational targets did this product **agree**, and where are they watched?

WHEN: the team has real numbers — not before. MUST NOT: invent `P95 < 200ms` or `99.9%` so the heading exists.

Headings, filled only with agreed values:

- Latency target (e.g. P95)
- Error-rate bound (e.g. 5xx)
- Availability / uptime target
- Which metric, under which condition, fires Warning vs Critical
- Auto-scaling policy **if this product has one**
- RTO / RPO
- Which services are critical vs not
- Which dashboard tracks each target

MUST: every number names the dashboard or alert that enforces it. A target with no signal is a wish.
MUST: runbook alerts **link** here; MUST NOT: two conflicting threshold lists.

---

## docs/known-issues.md

Question: what did we **knowingly** not finish? This file exists so a delivery cannot pretend everything is done.

WHEN: first production ship or handover, whichever is first. Update it when a known item is fixed or a new one is accepted.

Headings:

- Known bugs
- Technical limits
- Performance problems
- Workarounds
- Technical debt
- Follow-ups
- Priority of each item (what happens if it waits)

MUST: each item has an owner or a ticket if the project has tickets, and a priority. "We should fix it sometime" is not an item.
MUST NOT: delete a still-true item because the release notes look better without it.
WHEN: nothing is known.
HOW: one dated line that says so — not an empty file of headings.

---

## At ship: the living docs are the final architecture

Question: does `docs/` describe **the system that runs**, not the system we first drew?

MUST NOT: `docs/architecture/final.md`, `FINAL_ARCHITECTURE.md`, or a second overview "for handover". Reconcile the files that already exist.

WHEN: a version ships, or handover starts.
HOW: read and, if they drifted, update in the same change set:

- `docs/architecture/overview.md` — components, data flow, integrations in use **today**
- `docs/architecture/data-model.md` — nouns and relationships that exist
- `docs/architecture/api.md` — URLs and payloads the client actually gets
- `docs/plan/` — version checklists match what shipped in each version
- `docs/design/` — if a public frontend shipped, art direction matches what users see
- `docs/security.md` — if identity or personal data is in play

MUST: a sentence that is only true of the first design is removed or marked historical (an ADR is the place for "we used to").
MUST NOT: leave overview describing a service that was never built.

---

## docs/handover.md

Question: what does the next team need to take the product? Audience: the team receiving it.

WHEN: ownership is moving. This file is an **index** plus the few facts that live nowhere else (access, people, what is dangerous to touch).

Headings:

- System in a few lines (link overview)
- Critical components
- How to run it (link `docs/development/setup.md`)
- How to deploy it (link `docs/operations/deployment.md`)
- What is risky to change
- Known problems (link `docs/known-issues.md`)
- Technical debt (same file, or a heading that points at the debt section)
- External dependencies
- Access / ownership (roles and vault names, not secrets)
- People / teams
- Links to the docs that exist **today**
- Runbook / SLO (link)

MUST NOT: paste overview, runbook, and changelog into this file. Drift starts on the first edit.
MUST NOT [critical]: credentials in the handover.

---

## docs/lessons-learned.md

Question: what did **this organisation** learn? Not a technical spec.

WHEN: a project or a named phase closes — not after every slice.

Headings:

- What went well
- What went badly
- Decisions that turned out right
- Decisions that turned out wrong (link the ADR)
- Where time was lost
- Risks that should have been seen earlier
- What we will do differently next time

MUST NOT: a substitute for `known-issues.md` or the runbook. Those stay operational.
MUST NOT [critical]: personal data, credentials, or a dump of customer content from a postmortem.

---

## Same change set

WHEN: the code's architecture, data model, public JSON, authz, setup, or deploy path moved.
HOW: the matching file in this tree is part of the same slice, named in the plan ([06-plan.md](06-plan.md)).
MUST: name the doc change in the reply.
MUST NOT: "docs follow in a later PR". Later does not happen, and the next agent will read the old file.
MUST NOT: edit the document to match a shortcut you already took. Reconcile the document, then the code — same rule as the plan.

WHEN: a version is shipping or handover is the task.
HOW: `CHANGELOG.md`, `docs/known-issues.md`, and the living-docs reconcile above are in the slice. Runbook / SLO / handover only if their *Create when* row matches.

WHEN: you are about to touch a docs file no slice names.
HOW: stop. Update the plan, or say it is out of scope.

---

## Reading them

WHEN: [05-understand.md](05-understand.md) fires and `docs/` exists.
HOW: read the **one** file that matches the change — overview for system shape, data-model for a noun, `architecture/api.md` for a URL, the matching `docs/plan/{backend,frontend}/v*.md` for version scope, `docs/design/README.md` and the linked `<system-name>.md` for frontend layout/CSS/tokens, the ADR if a past choice is in play, security.md for authz/retention, `CHANGELOG.md` for a ship, `docs/known-issues.md` for leftover work, `docs/operations/runbook.md` / `slo.md` / `deployment.md` for production, `docs/handover.md` when ownership is moving. Not the whole tree.
MUST NOT: load this playbook file (09) unless you are creating or updating those docs, or the code change requires it (WHEN at the top).

---

## Done

MUST: before you stop —

- [ ] Anything that must survive this turn is in one of the three places, not in the chat
- [ ] No new `docs/` file was created before its row in *When each file exists*
- [ ] No secret landed in `README.md`, `CHANGELOG.md`, or `docs/`
- [ ] If architecture, data model, API, authz, setup, or deploy moved, the matching doc is in this change set
- [ ] A surviving "why" is an ADR (or a playbook `SCOPE` line), not only a plan bullet
- [ ] README still indexes; it did not absorb the technical document
- [ ] If this turn is a ship or handover: living docs match the running system; no `final-architecture.md`; known-issues names what was left undone
