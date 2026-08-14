# Coding Playbook

WHEN: creating or editing application code, **or** changing a playbook file because this project decided differently.
LOAD: this file first. Then the **stack** README that matches the files you are touching. MUST NOT: load every stack.

This folder is playbook, not application source. MUST NOT: copy it into `src/`.

---

## Not carved in stone

These files are a **starting map**, not scripture. They must fit **this** project.

MUST: if a rule here fights a real product constraint (stack, host, legal, an already-shipped shape), **adapt the rule to the project** — then **write that adaptation into the matching playbook file** in this folder (numbered file or Extra). The next agent reads the docs, not the chat.

MUST NOT: silently ignore a file and invent a parallel layout in `src/` while this folder still says the old thing.
MUST NOT: rewrite a file "for taste" when the project does not need a different shape.
MUST NOT: move Extra into day-one 01–16 because it might be useful later. Extra still loads only when that shape **already** exists.

How to change a playbook file: keep `WHEN` / `LOAD` / `MUST` / `MUST NOT`. Change the rule, the tree, or the Extra trigger. Leave a one-line reason in the file's `SCOPE` or at the Decide that moved. Do not leave two conflicting sentences.

---

## Stacks

1. Python + FastAPI API / workers — [python-fastapi-backend/README.md](python-fastapi-backend/README.md)
2. Next.js UI — [nextjs-frontend/README.md](nextjs-frontend/README.md)

No matching stack: stop. Do not invent a parallel layout from another stack. If this repo needs a new stack, add a folder and one line here — then write its map the same way.

## Extra (Python FastAPI)

Only if that API **already** has the shape. Map: [python-fastapi-backend/README.md](python-fastapi-backend/README.md) Extra section.

1. Isolated customers — [python-fastapi-backend/extra/01-multi-tenant.md](python-fastapi-backend/extra/01-multi-tenant.md)
2. Several deployable backends — [python-fastapi-backend/extra/02-microservices.md](python-fastapi-backend/extra/02-microservices.md)
3. In-product agent / team — [python-fastapi-backend/extra/03-agent-teams.md](python-fastapi-backend/extra/03-agent-teams.md)
4. Your library / package — [python-fastapi-backend/extra/04-packages.md](python-fastapi-backend/extra/04-packages.md)
5. WebSocket / SSE — [python-fastapi-backend/extra/05-realtime.md](python-fastapi-backend/extra/05-realtime.md)
6. Outbox — [python-fastapi-backend/extra/06-outbox.md](python-fastapi-backend/extra/06-outbox.md)
7. SSO (OIDC / SAML) — [python-fastapi-backend/extra/07-sso.md](python-fastapi-backend/extra/07-sso.md)
8. Search engine — [python-fastapi-backend/extra/08-search.md](python-fastapi-backend/extra/08-search.md)
9. Customer webhooks — [python-fastapi-backend/extra/09-webhooks.md](python-fastapi-backend/extra/09-webhooks.md)
10. Hide / retain / erase — [python-fastapi-backend/extra/10-retention.md](python-fastapi-backend/extra/10-retention.md)

## Extra (Next.js)

Only if that UI **already** has the shape. Map: [nextjs-frontend/README.md](nextjs-frontend/README.md) Extra section.

1. Tenant URL — [nextjs-frontend/extra/01-multi-tenant.md](nextjs-frontend/extra/01-multi-tenant.md)
2. Several Next apps — [nextjs-frontend/extra/02-apps.md](nextjs-frontend/extra/02-apps.md)
3. i18n — [nextjs-frontend/extra/03-i18n.md](nextjs-frontend/extra/03-i18n.md)
4. SSO (browser) — [nextjs-frontend/extra/04-sso.md](nextjs-frontend/extra/04-sso.md)
5. Realtime UI — [nextjs-frontend/extra/05-realtime.md](nextjs-frontend/extra/05-realtime.md)
6. Search UI — [nextjs-frontend/extra/06-search.md](nextjs-frontend/extra/06-search.md)
7. Uploads — [nextjs-frontend/extra/07-uploads.md](nextjs-frontend/extra/07-uploads.md)
8. Agent run UI — [nextjs-frontend/extra/08-agents.md](nextjs-frontend/extra/08-agents.md)
