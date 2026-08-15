# Glossary

> **For humans** — every term this playbook uses without stopping to explain it.

Read it when a rule file says something you cannot picture. Terms are grouped by where you meet them, not alphabetically, because that is how you will look them up.

**Türkçe:** [06-glossary.md](../tr/06-glossary.md) · **Index:** [for-humans/](../README.md) · **Agents read:** [agents/README.md](../../agents/README.md)

---

## The playbook itself

| Term | What it means |
|---|---|
| **Playbook** | This repo. Rules your team and your agents follow. Not application code — never copy it into `src/`. |
| **Stack** | One of the two rule sets: `python-fastapi-backend/` or `nextjs-frontend/`. Pick one per task. |
| **Numbered file** | `01`–`16` in each stack, ordered so `01` is what you decide first. An agent opens **one** of them per task, not all sixteen. |
| **Extra** | Optional rules under `extra/` (multi-tenant, SSO, search…). Loaded only when the product **already** has that shape — never to plan ahead. |
| **`WHEN:` / `LOAD:`** | Routing lines for agents: when this file applies, and what else to open. Humans can skim them. |
| **`MUST [critical]`** | Breaking it leaks a secret, exposes another user's data, or removes an authorization gate. Not a preference. |
| **`MUST` / `SHOULD`** | Unmarked `MUST` is the shape this playbook chose — consistency, not safety. `SHOULD` is a default you can deviate from with a reason. |
| **Slice** | One unit of work an agent does in one turn: one feature, one bug, one migration. Not "build the dashboard." |
| **Adapt in git** | When a rule doesn't fit your product, you change the playbook file with a one-line reason — so the decision lives in the repo, not in a chat log. |

---

## Frontend (Next.js)

| Term | What it means |
|---|---|
| **App Router** | The Next.js routing model this playbook targets — folders under `app/` become URLs. The older "Pages Router" is a different playbook. |
| **Server Component (RSC)** | A component that runs on the server and sends HTML. The **default**. It can read cookies and call the API directly, and it ships no JavaScript to the browser. |
| **Client Component** | A component marked `"use client"`, shipped to the browser so it can handle clicks and hold state. A cost, not a default — put the marker on the smallest leaf that needs it. |
| **Server Action** | A function marked `"use server"` that a form can post to. In this playbook it is a **thin** proxy to the API. It is also a public HTTP endpoint anyone can call, so it is never the place for a permission check. |
| **Hydration** | The browser re-running a component that the server already rendered. A "hydration error" means the two disagreed. |
| **Four states** | Every list, form, and panel needs loading, empty, error, and success — not just the success path. |
| **Token** | A named design value (`--surface`, `--space-2`) instead of a raw hex or pixel. Components use names; the hex lives in one file. |
| **Primitive** | A UI part with no product meaning: Button, TextField, Modal. Lives in `ui/`. If the filename contains a product noun, it is a feature instead. |
| **Feature** | One product noun and everything it means on screen — `features/orders/`. Pages compose features; features do not import pages. |
| **AI-slop** | The default look of a generative model: purple gradients, three equal cards, a dark sidebar with four stat tiles. Rejected on sight. |
| **Waterfall** | Requests that wait on each other instead of running together — usually a client fetch after a spinner, then another. |
| **`NEXT_PUBLIC_`** | Prefix that makes an environment variable visible to the browser. Baked into the bundle at build time and public forever. Never a secret. |

---

## Backend (FastAPI)

| Term | What it means |
|---|---|
| **Module** | One product ability under `modules/` — orders, billing, auth. Owns its rules. Deleting it removes something a user could do. |
| **Infra** | Adapters to systems you don't own the meaning of: Postgres, Redis, S3, Stripe. Holds no business rules. |
| **Repository** | The thin layer that reads and writes one table. No business rules, and it does not commit. |
| **Service** | Where a product rule lives ("cannot cancel if paid"). Called by HTTP routes and by workers alike. |
| **Worker** | A process that consumes a queued job outside the HTTP request — for slow, retryable, or fan-out work. |
| **Alembic revision** | One file describing a schema change, plus how to undo it. Schema history, replayed by a release job — the app never runs migrations at startup. |
| **Expand / contract** | Making a destructive schema change in two releases so old and new code can share the database during a rolling deploy. |
| **Cursor pagination** | Paging with "give me what comes after this record" instead of `page=2`. Stable when rows are being inserted underneath you. |
| **Idempotency key** | A client-supplied id that lets the API recognise a retried request and not charge twice. |
| **DLQ** | Dead-letter queue — where a job goes after it has failed too many times, so it can be inspected instead of lost. |
| **Outbox** | Writing "this event must be published" into the same database transaction as the data change, so the two cannot disagree. |

---

## Security and dependencies

| Term | What it means |
|---|---|
| **Authentication (authn)** | Who you are. Proving identity — logging in. |
| **Authorization (authz)** | What you may do. In this playbook it is always enforced on the API, never by hiding a button. |
| **404 for not-owned** | Returning "not found" rather than "forbidden" when a record exists but isn't yours — because "forbidden" confirms the record exists. |
| **HttpOnly cookie** | A cookie JavaScript cannot read. Where the session lives, so a script that gets injected still cannot steal it. |
| **XSS** | Injecting script into a page. React escapes text by default; `dangerouslySetInnerHTML` opts out of that protection. |
| **CSRF** | Another site making an authenticated request on your behalf. Blocked by `SameSite` cookies or a token the API requires. |
| **BFF** | Backend-for-frontend — a thin server layer that holds the cookie and talks to the real API on the browser's behalf. |
| **Lockfile** | The file recording exactly which version of every dependency, including transitive ones, was installed. Committed; CI installs **from** it. |
| **Pinning** | Fixing a dependency's version so builds are reproducible. Not the same as never upgrading. |
| **Advisory / CVE** | A published report that a specific package version has a known vulnerability. An outdated dependency with one is a finding, not backlog. |
| **Slopsquatting** | Registering package names that AI tools are likely to invent, so the hallucinated import installs an attacker's code. Why every new dependency gets checked against the registry. |
| **Supply chain** | Everything you install and everything it installs. Code that runs with your app's privileges without you writing it. |

---

**Next:** [01 Start here](01-start-here.md) · [02 How to prompt](02-how-to-prompt.md) · [03 Review agent code](03-review-agent-code.md) · [04 Pitfalls](04-pitfalls.md) · [05 Errors](05-errors.md) · [06 Glossary](06-glossary.md)
