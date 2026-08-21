# 07 · Verify, then report

WHEN: every turn, before you reply. Also the moment you are tempted to say "done", "fixed", or "should work".
LOAD: this file. Slice bookkeeping: [06-plan.md](06-plan.md) if the task has a plan.
RELATED: [04-errors.md](04-errors.md) when verification produced a new error.
SCOPE: what makes a claim true, when to stop, and what the reply contains.

You read your own diff and see what you meant, not what you wrote. That is why the evidence here is external: something ran, or something else read it.

---

## What is true, in order

WHEN: two sources disagree — the code, the playbook, this conversation, an earlier turn.
HOW: the higher one wins.

1. **The file on disk, read now.** Not remembered, not summarised.
2. **The playbook file** for that topic.
3. **What the user said this turn.**
4. **What was said in earlier turns.**

MUST NOT: follow stale chat over the current file. The user changed something, another agent ran, a rebase landed — the file knows and the transcript does not.
MUST NOT: trust your own earlier description of a file over the file. Your summary was true when you wrote it.
MUST: when the playbook and the user conflict, say which rule and let the user decide — except `[critical]`, where you say what it exposes first ([AGENTS.md](../AGENTS.md), *Rule strength*).

---

## Evidence: "written" is not "works"

WHEN: reporting a `Done when` item, a slice, or a fix.
HOW: each one gets exactly one of three verdicts.

**met** · **not met** · **blocked**

MUST NOT: a fourth state. "Should work", "probably fine", "looks correct" are all *not verified*, and saying so plainly is the honest version.

MUST: name the evidence next to the claim — the command and its result, the test that passed, the file you read back, the request you made. A claim with no evidence is reported as **written, not verified**, and you are the one who says it.

MUST: state what you did **not** verify. This is the most useful sentence in an agent's report, and the one most often missing.

---

## Use the strongest signal available

Ordered by how much they prove:

1. The app or endpoint actually running the path
2. Tests passing — the ones covering this change, not the suite's existence
3. The type checker
4. The linter
5. You, re-reading the diff

MUST: use the strongest signal the project already supports. If it has tests and you did not run them, that is a choice you must report.
MUST NOT: "I reviewed it carefully" as the only evidence. That is level 5, and level 5 is the one your failure mode lives in.
MUST NOT: write a test that cannot fail in order to produce evidence. A passing assertion of nothing is worse than no test (stack `14`).

---

## Stop conditions

WHEN: any of these is true, stop and hand back. Continuing costs more than asking.

- **The same fix failed twice.** A third attempt is not more effort — it is the wrong diagnosis. Report what you tried and what the errors were.
- **The change wants a file no slice names.** Update the plan or say it is out of scope ([06-plan.md](06-plan.md)).
- **Architecture, data model, public JSON, authz, setup, or deploy moved, and the matching product doc did not.** That slice is not met ([09-docs.md](09-docs.md)).
- **This turn is a ship or handover, and overview / data-model / API still describe the first design, or known-issues is missing while work was left undone.** That slice is not met ([09-docs.md](09-docs.md)).
- **A `[critical]` rule blocks the requested work.** Say which rule and what it exposes, offer the safe shape, and let the user decide ([AGENTS.md](../AGENTS.md)).
- **Only the user has the answer.** Finish everything that does not depend on it first ([05-understand.md](05-understand.md)).
- **`Done when` is met.** Stop. Stopping is the deliverable, not a missed opportunity.

MUST NOT: keep going because the turn feels short. Length is not value.
MUST NOT: fix code unrelated to the task while verifying. That is a new finding — name it, do not act on it ([03-anti-patterns.md](03-anti-patterns.md)).

---

## The reply

HOW: four parts, in this order, as short as they can honestly be.

1. **What changed** — the files, and what each one now does differently
2. **Evidence** — per `Done when` item: met / not met / blocked, with what proves it
3. **Not verified** — plainly
4. **Remaining** — what is left, what is blocked, and why

MUST NOT: paste playbook rules back ([02-turn.md](02-turn.md)). The user has the playbook.
MUST NOT: a summary longer than the change it summarises.
MUST NOT: bury a failure in the middle of a success list. If something did not work, it goes first.

---

## Done

MUST: confirm before replying —

- [ ] Every file I claim to have changed, I read after changing it or the edit tool confirmed it
- [ ] Each `Done when` item has met / not met / blocked, and none of them is a hedge
- [ ] The strongest available signal was used, or its absence is reported
- [ ] What I did not verify is stated
- [ ] Nothing outside the task was "fixed" along the way
- [ ] If a stop condition fired, I stopped and said why
