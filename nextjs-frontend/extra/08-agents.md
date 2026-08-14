# Extra 08 · Agent run UI

WHEN: the product shows an agent or team run (transcript, status, cancel).
LOAD: this file **and** 07, 08, 10. Backend: [python-fastapi-backend/extra/03-agent-teams.md](../../python-fastapi-backend/extra/03-agent-teams.md).
SCOPE: `features/agents` (or the noun that owns the run). MUST NOT: `app/agents` as a city of components.

Start: POST → `202` + `run_id` (09). Poll GET or Extra 05 stream. Four states (01). Tool names from events; MUST NOT: show raw model dumps or secrets (15, backend 03).

Cancel: POST the API; the button is not the control (12, 15).

MUST NOT: run the model in the browser. MUST NOT: `localStorage` as the transcript.

---

## Done

- [ ] Run id from the API; transcript from GET/stream
- [ ] Redacted events; cancel hits FastAPI
