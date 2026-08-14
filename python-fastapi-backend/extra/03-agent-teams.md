# Extra 03 · Agents (single and teams)

WHEN: the **product** runs one agent or a team of agents (LLM or scripted) as background work.
LOAD: this file **and** [09](../09-modules.md), [11](../11-workers.md), [15](../15-security.md). If the agent **executes** untrusted code or is a reusable kit: [Extra 04](04-packages.md). Not for coding agents that only edit this repo (they use 01–16).
SCOPE: where an agent is defined, where it runs, how a run is tracked, how a team is wired. MUST NOT: `src/agents/` as a third backbone.

An agent is not a Service class. It is a **job** that calls module services. The rule stays on `*Service` (09, 11).

---

## Single agent

### Where it is defined

If the work **is** one product noun ("summarise this order"): no `modules/agents/`. Add `workers/jobs/summarise_order.py` and methods on `OrderService`. Prompt/policy: helper file in `modules/orders/` when 01 splits (`summarise.py`).

If "agent" is itself a product (user starts a run, picks tools, sees a transcript): `modules/agents/` (09 day-one: router, schemas, service).

Definition (code, not a YAML city):

- `agent_kind` — string job type (`summarise_order`, `research_vendor`)
- tool allowlist — names of `*Service` methods it may call
- model/vendor settings — `config/` (which LLM) + `infra/` client (08). The module chooses **when** and **which** prompt.

MUST NOT: LangChain as a folder. MUST NOT: define tools that take SQL strings.

### Where it runs

Always a **worker** (11), never the HTTP request, never `BackgroundTasks`.

```
POST /v1/…                # or agents/runs
  → AgentService.start    # persist run, commit
  → infra/queue.publish
workers/jobs/<kind>.py    # session, AgentService.tick or OrderService.summarise
  → module services       # tools
  → commit events
  → ack
```

Long loops: each **step** is a job (or a step inside one job with a timeout from `config/`). Crash safety: state in Postgres, not RAM (11).

Scale: one `workers/runner.py` consuming several `agent_kind`s. [Extra 02](02-microservices.md) only if agent compute must fail/scale apart from the API.

### How work is tracked

Postgres (source of truth):

- `AgentRun` — `id`, `agent_kind`, `user_id`, `status` (`queued|running|succeeded|failed|cancelled`), `input` JSONB (ids, no secrets), timestamps.
- `AgentRunEvent` — `run_id`, `seq`, `kind` (`started|tool|model|failed|succeeded`), payload JSONB (redacted).

Write owner: `AgentService` (or `OrderService` if there is no agents module — then a `summaries` table on that noun). HTTP: `GET /v1/agents/runs/{id}` → 200; start → `202` + `run_id` (12).

Logs: `request_id` = `run_id` or `job_id` (04, 11). `audit` on start, each tool name, end (15). Client traces with `X-Request-ID` / `run_id`.

MUST NOT: Redis as the only run history (13). Cache may hold a live cursor.

Authz: payload `user_id`; every tool call goes through the **service** (15). MUST NOT: "the agent is the user."

---

## Agent teams

A team is an **orchestration**, not a new runtime. Members are single agents (same `AgentRun` rows).

### Where the team is defined

`modules/agents/` (required once a team is a product noun):

- `AgentTeam` — `id`, `name`, `graph` (ordered steps: `agent_kind` + join rule: all / any / map-reduce)
- or a frozen graph in code (`teams/support_triage.py`) until a second team needs a table

MUST NOT: each team as a microservice. MUST NOT: agents calling each other's worker files (`import jobs.b` — 11). Fan-out = `AgentService` publishes the **next** jobs after commit.

### How they run

```
AgentService.start_team
  → AgentTeamRun (status, team_id)
  → enqueue step 1 AgentRun(s)
worker finishes step
  → AgentService.on_member_finished(team_run_id, member_run_id)
  → if graph says so: enqueue next kinds
  → if terminal: mark team succeeded/failed
```

Parallel members: several jobs, same `team_run_id` in the payload. Join in `AgentService`, not in the runner.

Timeouts and cancellation: on the **team run**; members nack/cancel via the same service.

### How the team is tracked

- `AgentTeamRun` — `id`, `team_id`, `user_id`, `status`, timestamps
- Members: `AgentRun.team_run_id` FK. List events: team row + child runs + their events.

HTTP: `GET /v1/agents/teams/runs/{id}` returns team status + member run ids (schema in `modules/agents/schemas.py`).

### How layers plug in

Same chain as a single agent. No extra backbone folder.

- `http/` — auth, session, error JSON. No graph.
- `modules/agents` — graph, start, join, authz, persist
- `modules/<noun>` — tools (cancel, search, mail)
- `infra/queue` — bytes
- `infra/` LLM or sandbox — vendor verbs (08, [Extra 04](04-packages.md))
- `workers/jobs` — one kind per file; call `AgentService` / noun service

MUST NOT: a `TeamRuntime` in `http/`. MUST NOT: the LLM client in `shared/`.

---

## Done

- [ ] Agent runs in a worker; HTTP only starts and reads
- [ ] `AgentRun` (+ events) in Postgres; tools are `*Service` methods
- [ ] Team is a graph on `AgentService`; members are child runs; no job-to-job import
- [ ] Same 401/404/audit as a human caller (15)
