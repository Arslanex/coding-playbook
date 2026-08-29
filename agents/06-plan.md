# 06 · Plan, and work the plan

WHEN: the task needs more than one file, more than one turn, or touches a migration, auth, or a dependency. **And** every later turn of such a task.
LOAD: this file and [02-turn.md](02-turn.md). Analysis and design that feed the plan: [05-understand.md](05-understand.md).
RELATED: [07-verify.md](07-verify.md) — a slice is not done until it has evidence.
SCOPE: the plan file: when to write one, what is in it, and how it stays true. Loaded **every turn** of a planned task. A decision that must outlive the task is an ADR or a playbook line ([09-docs.md](09-docs.md)), not a plan that will be deleted.

Your context is not memory. A decision made three turns ago is not overwritten later — it is gone. Anything that must survive the task lives in a file, not in the conversation.

---

## When a plan is required

WHEN: the task matches any of these.
HOW: write the plan **before** the first edit.

- More than one file will change
- The work will not finish in this turn
- A migration, an auth change, or a new dependency is involved

MUST NOT: a plan for a one-line fix. A ritual that adds nothing is a ritual agents learn to skip, and the habit spreads to the plans that mattered.

---

## Where it lives

Default: `.agent/plan.md` in the **product** repo, git-ignored.

It is a file because a file survives the turn, can be re-read, and can be checked. It is git-ignored because it is scaffolding, not history — the decisions worth keeping end up in the product docs / ADRs ([09-docs.md](09-docs.md)), version checklists in `docs/plan/`, and in commit messages.

MUST NOT: the plan in the chat. That is the thing this file exists to prevent.
MUST NOT: the plan inside `coding-playbook/`. The playbook is rules; the plan is this task.

WHEN: the project wants it somewhere else (a tracked file, an issue, the tool's own mechanism).
HOW: fine — record that choice in this file's `SCOPE` line so the next agent finds it ([AGENTS.md](../AGENTS.md), *Not carved in stone*).

---

## Shape

```markdown
# Cancel an order from the UI
Stack: backend
Done when:
- [ ] PATCH /v1/orders/{id}/cancel returns 200 for an unpaid order
- [ ] paid order returns 409 with error_code, and a test covers it

## Design                      (only if 05's threshold fired)
- cancel rule -> modules/orders/service.py
- contract: {reason?} -> OrderResponse
- authz: service checks ownership; 404 for not-owned
- out of scope: partial refunds, cancel email

## Slices
- [x] 1. model + migration      loads: backend 06, 07
      evidence: alembic upgrade head passed; column present in test schema
- [ ] 2. service + HTTP         loads: backend 09, 10, 12
- [ ] 3. tests                  loads: backend 14

## Decisions
- status stored as text, not a DB enum — migration cost on every new value — revision a1b2

## Open
- does cancelling email the customer? (asked, waiting)
```

MUST: `Done when` items are observable. "Works correctly" is not an item; a status code, a passing test, a field name is.
MUST: every slice names the playbook files it loads. That is what keeps a later turn from loading all sixteen.

---

## The two rules that make a plan work

A plan written once and never re-read is worse than no plan, because you will act as if it is true.

**MUST: the first thing you do in a turn is read the plan.** Before the stack README, before any numbered file. You do not remember the plan — you read it. What is done, what is next, what is open.

**MUST: when reality and the plan disagree, update the plan first, then write code.** A plan that no longer describes the work is a lie, and next turn you will follow it. Reconcile, then act.

---

## Working through it

WHEN: a slice is in progress.
HOW: one slice per turn ([02-turn.md](02-turn.md)). Finish it, mark it, stop.

MUST NOT: start slice 3 while slice 2 is unchecked.
MUST NOT: mark a slice done without its evidence line ([07-verify.md](07-verify.md)).
MUST: if a slice turns out to be two things, split it **in the plan** — do not quietly do both.

WHEN: a new requirement arrives mid-task.
HOW: add it as a new slice. MUST NOT: fold it into the slice in progress — that is how a two-file change becomes an eleven-file diff nobody can review.

WHEN: you are about to touch a file no slice names.
HOW: stop. Either the plan is wrong (update it) or the change is out of scope (say so). MUST NOT: touch it and mention it afterwards.

---

## Decisions

WHEN: you decide anything a later turn could reasonably re-open — a name, a type, a library, a "we are not doing X".
HOW: write it in `## Decisions` **at that moment**, with the reason and where it landed. Not at the end; by then you will have forgotten the reason, which is the only part that matters.

MUST: if a decision contradicts a playbook rule, it goes in the **playbook file** too, with a one-line reason ([AGENTS.md](../AGENTS.md), *Not carved in stone*). The plan is deleted when the task ends; the playbook is not, and the next agent reads the playbook.

WHEN: the decision must survive this task as product truth — why this database, why this auth, "we are not doing X".
HOW: an ADR in `docs/architecture/decisions/` ([09-docs.md](09-docs.md)), written when the decision is made. Point the plan line at that file.

---

## Finishing

WHEN: every slice is checked and every `Done when` item has a verdict.
HOW: report ([07-verify.md](07-verify.md)) and stop.

MUST NOT: invent a slice 6 because the task felt short. Finishing is the deliverable.
MUST: anything left undone moves out of the plan and into your reply, named — not left checked-off-adjacent in a file the user may never open.

---

## Done

MUST: confirm before ending the turn —

- [ ] The plan was read at the start of this turn, not recalled
- [ ] The plan matches what the code now looks like
- [ ] Exactly one slice advanced, and it carries an evidence line
- [ ] Every file touched was named by a slice
- [ ] New decisions are recorded with their reason
- [ ] A decision that must survive this task is an ADR or a playbook line ([09-docs.md](09-docs.md)), not only a plan bullet
- [ ] Anything blocked is in `## Open` **and** in the reply
