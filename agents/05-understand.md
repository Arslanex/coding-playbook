# 05 · Understand before you build

WHEN: a task starts and you do not already know the code it touches — a new feature, a new noun, an unfamiliar file, or any change whose shape is not obvious from the prompt.
LOAD: this file and [02-turn.md](02-turn.md). Then [06-plan.md](06-plan.md) if the task needs a plan.
RELATED: [03-anti-patterns.md](03-anti-patterns.md) when the prompt is vague · [01-boundary.md](01-boundary.md) if the stack is still unclear.
SCOPE: what you establish **before** the first edit. Loaded once per task, not once per turn.

You write code from an assumption about the system, not from the system. This file closes that gap with actions, not intentions.

---

## Read before you write

WHEN: about to edit, create, or replace anything.
HOW: read the file you are editing. List the folder you are adding to. Read the signature you are calling. Read the schema before you query it.

WHEN: the product repo has `docs/` and the change touches a noun, a public URL, authz, how the system is shaped, a ship, handover, or how production is run.
HOW: read the **one** matching product doc (`docs/data-model.md`, `docs/api.md`, `docs/architecture/overview.md`, `docs/security.md`, the ADR in play, `CHANGELOG.md`, `docs/known-issues.md`, `docs/operations/runbook.md` / `deployment.md` / `slo.md`, or `docs/handover.md`). Not the whole tree. LOAD [09-docs.md](09-docs.md) only when you are creating or updating those files.

MUST NOT: write over a file you have not read in this session. Overwriting an unread file is the one edit that destroys work silently.
MUST NOT: rely on a summary of a file — yours or another agent's — when the file is one read away.

You know what the file *probably* contains. "Probably" is not a read.

WHEN: the change touches many files.
HOW: read the three the change actually edits, not the forty around them. If you genuinely need forty to proceed, the slice is too big — split it in [06-plan.md](06-plan.md) instead of reading your way through it.

---

## Analyse: four questions

WHEN: before any design or plan.
HOW: all four must have an answer. An unanswered one is either a thing to check, or a question for the user (below).

1. **What already exists?** The adjacent code that does something similar. Extend or call it.
2. **What is the contract?** Response shape, DB columns, prop types, the `error_code` the caller branches on.
3. **What breaks if I am wrong?** Reversible in an edit, or a migration with data in it? This sets how much certainty the change deserves.
4. **Where does this belong?** The Decide list in the stack's structure file — not your own taxonomy.

MUST NOT: start from an empty file when an adjacent one exists. A second parallel implementation is the most common structural defect and the hardest to unwind later.
MUST NOT: infer a contract from a variable name. Read the schema, the Pydantic model, or the Zod schema.

---

## Assume, check, or ask

WHEN: something you need is not established.
HOW: stop at the first match. These are three different actions, not three levels of politeness.

**1. Check** — the answer is in the repo, a file, a registry, a type, or one command.
→ Get it. Do not ask, do not assume. Cost is seconds.
MUST NOT: ask the user what the repository can answer. That is not caution; it is a skipped read, and it is the single most irritating thing an agent does.

**2. Assume and state** — the answer is not available, but every reasonable answer leads to the same work, or being wrong is cheap to redo.
→ Write the assumption in **one line**, then proceed. The line goes in the plan ([06-plan.md](06-plan.md)) and in your reply.
MUST NOT: a silent assumption. An unstated assumption is indistinguishable from a mistake.

**3. Ask** — different reasonable answers lead to **materially different work**, or being wrong is expensive or irreversible: a schema shape, an auth model, a public URL, deleting or overwriting data, a product rule only the user knows.
→ Ask, then wait for that specific answer.

MUST: batch questions. One round, all of them together — not one question, then another, then another.
MUST: before asking, finish everything that does not depend on the answer. A blocked question with nothing delivered is the expensive kind.
MUST NOT: assume on anything irreversible. The cost is asymmetric; treat it that way.

---

## Design the system, at a threshold

WHEN: the change does any of these — otherwise skip this section and go build.

- Adds a new noun: a table, a `features/<noun>/`, a `modules/<capability>/`, an endpoint family
- Touches **both** stacks
- Changes a contract other code already depends on
- Introduces state that outlives one request: a queue, a cache key, a cookie, a stored file

HOW: write five to ten lines. Not a document. It states:

- **Nouns and their homes** — each one names the file or folder it lands in
- **The contract** — request/response shape, or the props and types crossing the boundary
- **Who owns the write, who owns authz** — in this playbook always FastAPI, never the UI
- **What changes in the database** — and whether it needs expand/contract (backend `07`)
- **What is explicitly out of scope** — the line that stops the next turn from growing

MUST: the design goes into the plan ([06-plan.md](06-plan.md)), not into the chat. Chat is not a place decisions survive.
MUST NOT: an abstract design. Every decision names a path.
MUST NOT: invent a shape the playbook already decided. Where a file goes, what a module owns, how errors are shaped — that is the numbered stack file's job, and yours is to apply it.
MUST NOT: design past the current task. A design for features nobody asked for is scope creep with a diagram.

---

## Analysis is not the work

WHEN: the four questions have answers and the design (if required) is written.
HOW: build. Now.

MUST NOT: spend the turn reading. The bias runs both ways — an eager agent skips this file entirely, a cautious one never leaves it. Both fail the user.

---

## Done

MUST: before the first edit, confirm —

- [ ] Every file being edited was read in this session
- [ ] The four questions have answers, not guesses
- [ ] Anything checkable was checked, not asked and not assumed
- [ ] Assumptions are written down, in one line each
- [ ] Questions, if any, were asked once and together
- [ ] If `docs/` exists for this change, the matching product doc was read ([09-docs.md](09-docs.md))
- [ ] If the design threshold fired, the design is in the plan and names paths
