# Reviewing agent-written code

> **For humans** — whoever approves the pull request. You are merging code you did not write.

Each stack has a `15-security.md` with an "Agent pass (PR)" checklist. That is the agent checking its own work. This page is **you** checking the agent, and the two are not the same job: the agent verifies the rules it remembered, and the failures below are the ones it did not notice it was making.

You do not need to read every line. You need to read the **right** lines, in the right order.

**Türkçe:** [03-review-agent-code.md](../tr/03-review-agent-code.md) · **Index:** [for-humans/](../README.md) · **Agents read:** [agents/README.md](../../agents/README.md)

---

## Read the diff in this order

Stop at the first thing that is wrong. A bad answer at step 1 makes steps 3–5 irrelevant.

**0. The plan, if there is one.** On a task spanning more than one turn, the agent keeps a plan at `.agent/plan.md` — task, done-when, slices, and the decisions it made along the way. Read it before the diff. Comparing what the agent *said it would do* against what it did is faster than reading code, and the `## Decisions` section tells you things a diff never shows: why a type was chosen, what was ruled out. If the plan and the diff disagree, that is your finding.

**1. The file list, before any code.** Not what changed — *which files* changed. Does the set match what you asked for? A request to add one field that touches eleven files is the finding, and you have it in five seconds.

**2. The data model and any migration.** Schema is the hardest thing to walk back. One release later, a wrong column has data in it. Check that the migration does one logical change, that `downgrade()` is filled, and that old code can still boot on the new schema.

**3. The boundary.** Where does this code decide who is allowed to do something? For this playbook the answer is always the API, never the UI. A permission check that appears in a React component or a Server Action is a defect no matter how correct it looks.

**4. The states nobody asked about.** Loading, empty, error. Agents generate the success path by default. If the diff adds a list and only shows the list, it is unfinished.

**5. The tests.** Read what they *assert*, not that they exist. Then read them again asking: would this test fail if the feature broke?

---

## Eight failure modes specific to agent-written code

Each one has a signal you can look for directly, without reading the whole diff.

### 1. Scope creep

**Signal:** files in the diff you did not mention, "while I was there" in the description, formatting changes mixed with logic.

Ask for the unrelated part to be reverted, not explained. The playbook's own rule is one slice per turn ([agents/02-turn.md](../../agents/02-turn.md)). A diff that quietly does two things is one you cannot revert cleanly when half of it turns out to be wrong.

### 2. A package that does not exist

**Signal:** any new line in `package.json` or `pyproject.toml`.

Agents do not reliably know which package names are real, and attackers register the plausible-sounding ones that get invented. Open the registry page for every added dependency. Check the last release date and the linked repository — a package that is real but abandoned is a different problem with the same fix: don't.

Also worth asking: does the platform already do this? A date format, a debounce, a UUID is not a dependency.

### 3. A rule was adapted in code but not in the playbook

**Signal:** the code does something the playbook file says not to, and the playbook file is unchanged in the diff.

This is the one that rots the repo. The playbook's contract is that a decision lives in git, not in a chat log ([01 Start here](01-start-here.md)). If the rule genuinely had to bend for your product, the playbook file changes **in the same PR**, with a one-line reason. If it did not have to bend, the code changes instead. What must not happen is the two drifting apart, because the next agent reads the file and re-introduces the thing you just accepted.

### 4. Tests that cannot fail

**Signal:** a test with no assertion, a snapshot of a whole page, a mock that returns exactly what the assertion checks.

A snapshot test on generated UI is worse than no test: it locks in whatever the agent produced, including the parts you would have rejected. Ask what behaviour the test protects. If the answer is "it renders," it protects nothing.

### 5. Happy path only

**Signal:** a new screen with no empty state; a `fetch` with no error branch; a form that assumes the API said yes.

### 6. Authorization in the interface

**Signal:** `if (user.role === …)` in a component, a hidden button, a disabled control described as a permission.

A hidden button is not a control — curl is. The check must exist on the API, and the UI hiding it is a nicety on top. Verify the API rejects the call, not that the button is gone.

### 7. A secret or a per-environment value in the browser bundle

**Signal:** a new `NEXT_PUBLIC_*` variable.

Two separate defects share this signal. If it is a secret, it is now public forever and rotating it means rebuilding every deploy that shipped it. If it is merely environment-specific, it was baked in at build time and will be wrong the moment the image is promoted from staging to production. Neither is fixed by changing the value on the host.

### 8. A migration that cannot be undone or was edited after shipping

**Signal:** `downgrade()` with `pass` in it, or a change to a revision file that is already in `main`.

---

## When to push back, and when to let it go

The playbook marks its rules at three strengths, and they mean something for you as a reviewer too.

| Marking | What it protects | What you do |
|---|---|---|
| `MUST [critical]` | A secret, another user's data, or an authorization gate | Do not merge. This is not a preference. |
| `MUST` (unmarked) | The shape this playbook chose — consistency, not safety | Merge if the product genuinely needs the exception; then update the playbook file in the same PR |
| `SHOULD` | A sensible default | Reviewer's judgment. Not worth a round trip on its own. |

The failure worth naming: treating an unmarked `MUST` as if it were critical. If you block a PR over folder naming with the same force you would block a leaked key, people stop trusting the distinction, and the next real one gets waved through.

---

## The sixty-second version

- [ ] The changed-file list matches what I asked for
- [ ] Every new dependency exists, is maintained, and is actually needed
- [ ] Schema change: one logical change, `downgrade()` filled, old code still boots
- [ ] Every permission decision is on the API, not in the UI
- [ ] Loading, empty, and error states exist for anything new on screen
- [ ] Tests would fail if the feature broke
- [ ] No new `NEXT_PUBLIC_*` holding a secret or an environment-specific value
- [ ] If a playbook rule was bent, the playbook file changed in this same PR

---

**Next:** [01 Start here](01-start-here.md) · [02 How to prompt](02-how-to-prompt.md) · [03 Review agent code](03-review-agent-code.md) · [04 Pitfalls](04-pitfalls.md) · [05 Errors](05-errors.md) · [06 Glossary](06-glossary.md)
