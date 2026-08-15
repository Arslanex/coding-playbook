# Vibe coding pitfalls

> **For humans** — teams building real, shipped software with AI coding tools.

Vibe coding is seductive: you describe something in plain language, working code appears, you describe a change, it happens. The loop from idea to execution shrinks to minutes.

Then something breaks and you do not know why.

The speed that makes it powerful is the same speed that makes it dangerous. Bad decisions happen faster, shortcuts compound faster, and generated code *looks* professional — so problems hide longer than in code you struggled to write yourself.

These are not theoretical risks. They are the mistakes that consistently kill projects. All are fixable if you catch them early. This playbook does not replace discipline — it **channels** it: small prompts, explicit layers, config and security rules, git as the source of truth.

**Türkçe:** [04-pitfalls.md](../tr/04-pitfalls.md) · **Index:** [for-humans/](../README.md) · **Agents read:** [agents/README.md](../../agents/README.md)

---

## 1. Prompting like you're texting a friend

**Bad:** “Build me a dashboard.”

Five words. The agent chooses metrics, layout, data source, and roles. You get something that looks like a dashboard but matches none of your real constraints — then spend more time undoing assumptions than you saved.

**Fix:** Specificity and playbook routing. Name the stack, files, entities, and done criteria. If the prompt could describe a thousand apps, it is too vague.

```text
Stack: nextjs-frontend
Task: Customer success dashboard — monthly churn, 12-month NPS trend, at-risk accounts by health score.
Playbook: 01-design + 10-features + 07-data + 09-api-client
Done when: four UI states (01); data from FastAPI /v1/… only; no client-side secrets
```

More templates: [02 How to prompt](02-how-to-prompt.md).

---

## 2. Building everything in one giant prompt

**Bad:** One 500-word message with every feature, page, and edge case.

Past a complexity threshold the model makes silent tradeoffs: features merge, interactions drop, layers blur (HTTP logic in UI, DB access from Next.js).

**Fix:** Iterative building aligned with playbook slices.

1. Core workflow (one module / one feature package)
2. Settings, then payments, then admin — **separate prompts**
3. Each step: working checkpoint in git before the next prompt

Chain: model + migration → service + API → worker (if needed) → UI feature → tests. See [02 How to prompt § Biggest pain points](02-how-to-prompt.md).

---

## 3. Skipping version control

**Bad:** Accept AI diffs quickly with no commits. Feature X breaks; something unrelated regresses; you cannot return to last good state.

**Fix:** Commit early, commit often. Every prompt that produces a **working** result is a checkpoint. Before a risky prompt: commit. When playbook rules change to match a product decision, **commit the playbook file too** — the next agent reads git, not chat ([README.md](../../README.md) “Not carved in stone”).

---

## 4. Not reviewing generated code

**Bad:** Polished formatting creates false confidence. Research consistently shows: many AI solutions are functionally correct but a large share fail security review (auth, injection, secrets, validation).

**Fix:** Treat every agent output as a **draft**.

- Read the diff, especially auth, input validation, env handling, SQL/query construction
- Run [python-fastapi-backend/15-security.md](../../python-fastapi-backend/15-security.md) or [nextjs-frontend/15-security.md](../../nextjs-frontend/15-security.md) before merge
- If you cannot review code yourself, pair with someone who can or use automated security checks on PRs

Working code is not safe code.

---

## 5. Ignoring the data model

**Bad:** “Build a project management tool.” The agent invents entities and relations. You build six features on the wrong skeleton; fixing the model cascades through migrations, APIs, and UI.

**Fix:** Describe **data before features**.

- Entities, relations, constraints, status enums
- Backend: [06-database.md](../../python-fastapi-backend/06-database.md) + [07-migrations.md](../../python-fastapi-backend/07-migrations.md) in the **same** prompt as the model change
- Connecting to an existing DB: paste or `@` schema; do not let the agent guess table names

Example constraint block:

```text
Projects have many Tasks. Task → one assignee, many watchers.
Status: todo | in_progress | review | done. Priority: P0–P3.
```

---

## 6. Trusting the AI's security defaults

**Bad:** Generated code optimizes for “works on my machine.” Common failures: hardcoded secrets, missing validation, SQL injection, secrets in client bundles (`NEXT_PUBLIC_`), permissive authz.

**Fix:** Security rules live in the playbook **independently** of whatever the agent wrote today.

| Area | Playbook |
|------|----------|
| Backend secrets, authz, JWT | [13-identity-security.md](../../python-fastapi-backend/13-identity-security.md), [15-security.md](../../python-fastapi-backend/15-security.md) |
| Frontend cookies, XSS, env | [12-auth.md](../../nextjs-frontend/12-auth.md), [15-security.md](../../nextjs-frontend/15-security.md) |
| Config (no leaked keys) | [03-config.md](../../python-fastapi-backend/03-config.md), [04-config.md](../../nextjs-frontend/04-config.md) |

Prompt explicitly: “No secrets in client code; all limits in Settings / lib/env.ts; authz stays on FastAPI.”

---

## 7. Building too much before validating with users

**Bad:** Vibe coding removes the old cost of building the wrong thing. You ship eight features in a day; users only needed the first one, differently.

**Fix:** Minimum useful workflow first. Real users, then the next prompt. Playbook Extra topics (SSO, multi-tenant, agents, search) only after the shape is **shipped and validated** — not because the doc exists ([01 Start here](01-start-here.md)).

---

## 8. Context window overflow

**Bad:** Long threads “forget” requirements. Login breaks though you did not touch auth. Random regressions.

**Fix:** Fresh chat per module or feature slice. Re-attach: short decision summary (3 lines) + minimal playbook `@` files + current app paths. Do not rebuild the whole app in one thread.

| Tooling | Helps | Does not eliminate |
|---------|-------|---------------------|
| Codebase indexing (Cursor, etc.) | Finds files without pasting everything | Long chat drift |
| Playbook `LOAD:` routing | Limits rules per task | Mega-prompts |
| Decision doc in repo | Paste into new chats | Need for new threads |

Full playbook: [02 How to prompt § Context bloat](02-how-to-prompt.md).

---

## 9. No error handling or edge cases

**Bad:** Happy path only — full forms, APIs always 200, DB always returns rows. Production: timeouts, empty results, validation failures.

**Fix:** After happy path works, **second prompt** for defense.

```text
Task: Error paths for [feature] — API 4xx/5xx, empty list, network timeout, invalid form fields.
Playbook: backend 05-errors + 12-api OR frontend 01-design (four states) + 08-forms
Done when: user-visible message; no silent failure; logged server-side with request_id
```

Backend error JSON: [05-errors.md](../../python-fastapi-backend/05-errors.md), [12-api.md](../../python-fastapi-backend/12-api.md). UI states: [01-design.md](../../nextjs-frontend/01-design.md).

---

## 10. Treating AI output as finished code

**Bad:** The meta-mistake. Deploy without review, scale without tests, extend without understanding. Looks good until it does not.

**Fix:** Match generation speed with review discipline.

```
AI writes → you review diff → you test → AI fixes → commit checkpoint → next slice
```

Playbook = contract for **both** humans and agents. When reality diverges, update the playbook file with a one-line reason — do not leave chat as the only source of truth.

---

## Quick checklist before you ship

- [ ] Prompt was specific (stack, files, playbook files, done when)
- [ ] Git checkpoint before and after the change
- [ ] Diff reviewed; security files checked for this stack
- [ ] Data model explicit; migrations if backend changed
- [ ] No secrets in frontend env; config in Settings / `lib/env.ts`
- [ ] Error and empty states handled
- [ ] Not “everything in one thread”

---

**Next:** [01 Start here](01-start-here.md) · [02 How to prompt](02-how-to-prompt.md) · [03 Review agent code](03-review-agent-code.md) · [04 Pitfalls](04-pitfalls.md) · [05 Errors](05-errors.md) · [06 Glossary](06-glossary.md)
