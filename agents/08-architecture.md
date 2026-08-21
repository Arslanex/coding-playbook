# 08 · Architecture and build order

WHEN: a new service or app is starting, the repo is empty or nearly so, or the user describes a **product** rather than a change ("build me an order system", "we need a CRM").
LOAD: this file and [05-understand.md](05-understand.md). The slices it produces go in [06-plan.md](06-plan.md).
RELATED: [01-boundary.md](01-boundary.md) for which stack · [03-anti-patterns.md](03-anti-patterns.md) before creating many files.
SCOPE: the **project** — what is agreed before the first line, what order things get built in, when the agreement is rewritten. One feature's path through the layers is [02-turn.md](02-turn.md), not this file. Day-one docs: `docs/data-model.md` and `docs/architecture/overview.md`; the rest of the product `docs/` tree is [09-docs.md](09-docs.md).

---

## Talk first: six questions

WHEN: the user describes a product, not a change.
HOW: ask all six in **one** message ([05-understand.md](05-understand.md)), short, in this order. Each produces one thing.
MUST NOT: write a file before all six have answers. A wrong answer here is not a bug, it is a rewrite.
MUST NOT: six rounds of question-and-confirm. One round of questions, one confirmation of the synthesis (below). A discovery interview is not a deliverable.

**A · Nouns and ownership**
HOW: ask what the things are, which belongs to which, and how each is created, changed, and ended.
→ `docs/data-model.md`.

**B · Actors and isolation**
HOW: ask who uses this, whether roles exist, whether data is shared / per-user / per-organisation, and what a user must **not** see.
→ `docs/architecture/overview.md`.
MUST: ask before the first table. Ownership and isolation reach into every query and every service; adding them later means touching all of them.

**C · Coverage**
HOW: ask whether one HTTP API and one web UI cover everything needed — or whether there is also a mobile app, a CLI, a scheduled data job, a bot, a desktop or embedded target.
→ decides whether this playbook covers the work at all. Handled below.

**D · Scale**
HOW: ask how many users or systems at once, how much data, and whether this is an MVP or must be production-ready.
→ decides when infrastructure enters (build order step 5). MUST NOT: skip it and then guess whether a queue is needed.

**E · Fixed constraints**
HOW: ask what already exists and cannot change — a database, an internal service, a host, a compliance rule.
→ `docs/architecture/overview.md`.
MUST: separate "already exists" (a constraint) from "would prefer" (a choice you may argue with).
MUST NOT: ask which language or framework. This playbook is the stack; the open questions are the database and the outside systems.

**F · First slice, and what is out**
HOW: ask for the smallest slice a real person could use end to end — **and** what is explicitly not in it.
→ the slice list in [06-plan.md](06-plan.md), and the out-of-scope line in `docs/architecture/overview.md`.
MUST NOT: accept "the whole product". MUST NOT: a login screen with nothing behind it.

---

### When an answer does not come

WHEN: the user answers vaguely, or says "you decide", "I don't know", "later".
HOW: offer **two concrete options** with their consequence in one line each, and let them pick. A choice between two named things is answerable; an open question is not.
MUST NOT: repeat the same question in different words.

WHEN: they still will not choose, or the question is B.
HOW: state the default in one line and proceed. For B: single tenant, one role, every record owned by its creator, ownership checked in the service.
MUST: write that default into `docs/architecture/overview.md` as a decision. It is one. If a later agent would re-open it, it also becomes an ADR ([09-docs.md](09-docs.md)).

MUST NOT: invent nouns the user did not mention ([03-anti-patterns.md](03-anti-patterns.md)).

---

### C · when the answer is not "one API and one web UI"

Stop at the first that matches.

1. **One API, one web UI** → this playbook covers it. Continue.
2. **A second deployable of the same kind** — another backend, an admin UI on its own host.
   → Ask whether that is true **today** or on a roadmap. On a roadmap: build one, and say so. True today: `extra/02-microservices.md` or `extra/02-apps.md`, and those files' own gates still apply.
3. **Something this playbook has no stack for** — mobile, CLI, desktop, embedded, a data pipeline, an ML training job, a bot.
   → **Stop.** MUST: say plainly that no stack exists for it, and give the two ways forward: leave it out of scope for now, or add a stack folder and write its map ([AGENTS.md](../AGENTS.md), *Stacks*).
   MUST NOT: bend `python-fastapi-backend` or `nextjs-frontend` onto it. A CLI is not a FastAPI app with the routes removed.
   MUST NOT: discover this at file-writing time. It is question C for exactly this reason.

---

### Confirm once, then build

WHEN: all six have answers.
HOW: write the synthesis back in one short message — the nouns, the isolation model, what is in the first slice, what is out, and every default you chose on their behalf. Ask for one confirmation.
MUST: name the assumptions explicitly ("I assumed X — correct?"). MUST NOT: a silent assumption ([05-understand.md](05-understand.md)).
MUST NOT: start writing files before that confirmation. MUST NOT: ask for a second one.

---

## The two documents

WHERE: the **product** repo — `docs/data-model.md` and `docs/architecture/overview.md`. Committed, unlike the task plan: they outlive the task and a human reviews them. The rest of `docs/` (setup, ADRs, API, security, deploy) is created when that shape exists — [09-docs.md](09-docs.md). MUST NOT: scaffold those files on day one.

`docs/data-model.md` — nouns, fields, relationships, who owns each row. Enough that a migration can be written from it.
MUST NOT: a field the product has no use for yet.

`docs/architecture/overview.md` — actors and roles, isolation model, who owns writes and authz (here: FastAPI, always), the outside systems in use **today**, what is out of scope. Ten to thirty lines on day one; it grows as components exist ([09-docs.md](09-docs.md)).
MUST NOT: a diagram of a system nobody asked for.
MUST NOT: a second file at `docs/architecture.md`.

MUST: both exist before the first migration, and the user has seen them before the first module.
MUST NOT: keep either in the conversation. The next agent reads the repo.

---

## Build order

WHEN: the documents exist and the first slice is agreed.
HOW: this order. Each step exists because the next imports it — out of order means inventing a local version and unwinding it later.

**0 · Skeleton** — what everything else assumes.
WHERE: `pyproject.toml` + lockfile + pinned Python (02) · `config/` + `.env.example` (03) · `shared/errors/` (05) · `shared/logging/` (04) · `main.py` + `http/router.py` + `http/deps.py` (10) · `infra/db/session.py` (06) · Alembic wired, chain empty (07) · a test database and one passing test (14).
MUST: one slice, not seven. It is boring and fast, and every later step is cheaper for it.
MUST NOT: a business rule anywhere in step 0.
MUST NOT: a `Dockerfile`, compose file, or CI workflow unless the user asked ([01-boundary.md](01-boundary.md)). If the test database needs a container the project lacks, give the command and stop.

**1 · Data model** — the core.
WHERE: models for the nouns the first slice needs (06) and the first migration (07), same turn. `docs/data-model.md` becoming code.
MUST NOT: tables for nouns beyond the first slice. An unused table is a migration you will undo.

**2 · Domain** — the product itself.
WHERE: `modules/<capability>/` — the service holding the rules, calling repositories (09).
MUST: write the service **before** the route, so the rule cannot end up in the router.

**3 · Edge** — how the outside reaches it.
WHERE: router, request/response schemas, mount, `error_code` mapping (10, 12, 05). The first slice is now callable.

**4 · Identity** — when there is something worth protecting.
WHERE: auth module, `get_current_user` on protected routes, ownership checks in services, 404 for not-owned (13, 15).
HOW: build the first capability unauthenticated to learn its shape, then add identity **before the second**. Earlier means permissions for nouns that do not exist; later means retrofitting ownership into every service.
MUST NOT: ship a route that touches user data before this step is done.

**5 · Infrastructure** — pulled, not pushed.
WHERE: `infra/cache/`, `infra/queue/`, `infra/storage/`, `workers/` (08, 11).
MUST: each enters only when a requirement demands it — work that must survive a crash, a rate limit needing shared state, a file that must not live in Postgres.
MUST NOT: Redis, a queue, or object storage in step 0 because the architecture has them. This is the most common way a day-one backend becomes unmaintainable.

**6 · Frontend consumes it.**
WHERE: frontend `01-design` → `03-file-structure` → `10-features` → `07-data` / `08-forms` → `12-auth`.
MUST NOT: build UI against an endpoint that does not exist yet. The contract drifts and the UI encodes a guess.

**7 · Harden.**
WHERE: security pass (15), performance where it is measurably slow (16), tests filled to the four states (14).
MUST NOT: an Extra shape the product does not already have.

WHEN: the first slice ships and the next capability begins.
HOW: re-enter at step 1 or 2. Steps 0 and 4 happen once.
MUST NOT: return to step 0 to "improve the foundation" without a requirement forcing it.

---

## When the agreement changes

WHEN: any of the four below. The document changes **first**, then the code.

WHEN: a noun appears that the data model does not have.
HOW: `docs/data-model.md`, then the model, then the migration.

WHEN: the actor or isolation model changes — roles appear, tenants appear, sharing appears.
HOW: `docs/architecture/overview.md` first.
MUST: treat it as its own task with its own plan. It touches authz in every service. MUST NOT: fold it into a feature.

WHEN: a new outside system is required.
HOW: `docs/architecture/overview.md` first, with the requirement that pulled it in written next to it (step 5).

WHEN: the code has a shape the documents do not describe.
HOW: reconcile the document, then act — same rule as the plan ([06-plan.md](06-plan.md)). A document that stopped being true is worse than none; the next agent will believe it.

MUST: name any change to either document in your reply. The user agreed to the old one.
MUST: if a "why" changed, the ADR is in the same change set ([09-docs.md](09-docs.md)).
MUST NOT: edit the architecture to match a shortcut you already took. That is how a document becomes fiction.
MUST NOT: expand either document with what the product might need later. They describe what exists and what is committed to next.

---

## Done

MUST: before the first file of a new project —

- [ ] All six questions have answers — asked once, in one message
- [ ] Question C answered: the work fits one API + one web UI, or the mismatch was raised before any file
- [ ] The synthesis was confirmed by the user, assumptions named
- [ ] `docs/data-model.md` and `docs/architecture/overview.md` exist and the user has seen them
- [ ] Defaults chosen on the user's behalf are written down as decisions
- [ ] The first shippable slice is named, and it is genuinely the smallest one
- [ ] The plan's slices follow the build order above ([06-plan.md](06-plan.md))
- [ ] No infrastructure in the plan that no requirement pulled in
