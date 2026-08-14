# 16 · Performance and resources

WHEN: a pool, timeout, query that might return many rows, cache "for speed", worker concurrency, a slow request, an unbounded loop, or "should we split a service to go faster?".
LOAD: this file **and** [03-config.md](03-config.md) — every cap on this page is a settings field, not a literal.
RELATED: 06, 08 (where the pool is opened) · 11 (jobs) · 12 (lists) · 07 (indexes) — open only if the task is also that topic.
SCOPE: finite budgets (connections, memory, CPU, file descriptors, vendor rate) and how a use case spends them. Not [Extra 02](extra/02-microservices.md). Not a new `src/` folder.

A resource without a cap is a leak. Speed comes from a smaller use case, not from a new city in the tree.

---

## Decide: make it cheaper

Stop at the first yes. MUST NOT: skip to Redis, a replica, or a second microservice.

1. Unbounded result, Python-side filter, loop of `get_by_id`, or a list with no `LIMIT` / cursor?
   → Fix the query (06, 12). Add the index the `WHERE` / `ORDER BY` uses (07).
2. The client must not wait (mail, export, vendor, agent loop, ML, PDF)?
   → Job after `commit()` (11). HTTP stays a short write + `202`.
3. The session is open while the process waits on HTTP/S3/LLM?
   → Close the unit of work first (or do not open it until the wait is over). Vendor I/O does not hold a Postgres connection.
4. Same reconstructible DTO is read far more than it is written?
   → Module chooses key + TTL, `infra/cache/` stores bytes (13). MUST NOT: cache to hide N+1.
5. CPU-bound work on the API event loop (hash a large file, train, render)?
   → Worker process (11), or [Extra 04](extra/04-packages.md) package **called from** that job. MUST NOT: `ProcessPool` inside `http/`.
6. One replica is at a measured cap (CPU, pool wait, queue depth)?
   → Scale **this** process (more pods, or split worker kinds in 11). MUST NOT: [Extra 02](extra/02-microservices.md) "for performance".

MUST NOT: add `infra/cache/` because a GET exists. MUST NOT: split services so two pools "feel safer". MUST NOT: `BackgroundTasks` so the request returns faster (11).

---

## Bound everything

Every number below lives in `config/`. MUST NOT: magic `20` / `100` in `session.py`, a job file, or a repository.

MUST cap:

- DB pool (`pool_size` + `max_overflow`) per process
- Worker in-flight jobs (semaphore, 11)
- Queue prefetch ≤ that semaphore
- HTTP client: connect timeout, total timeout, max connections
- Redis: socket timeout + pool
- List `limit` (12, default 100)
- Upload bytes (15)
- Queue payload size (ids, not file bytes — 11)
- Outbound retries (bounded; honor `Retry-After`)

MUST NOT:

- `list_all` / `select()` with no `LIMIT` on a user-facing path
- `asyncio.create_task` in a loop with no semaphore
- Fan-out 10k publishes in one request; batch or a job that pages
- Load a whole table into Python to `.filter` in memory
- Sync `requests` / `time.sleep` inside `async def`

---

## Pool budget (one process)

API (`main.py`) and worker (`workers/runner.py`) are **two** processes. Each has its own engine and pool (06). Uvicorn/Gunicorn **workers** are more processes: total Postgres connections ≈ `process_count × (pool_size + max_overflow)`.

MUST: size the pool for **in-flight use cases that hold a session**, plus a small headroom. Worker: `pool_size >= concurrency` (one session per in-flight job, 11). HTTP: pool ≥ concurrent requests that open `get_session`.

MUST NOT: `pool_size=100` "in case". A large pool waits on Postgres, not on Python.

MUST: `pool_pre_ping=True`, `dispose()` on shutdown (06). MUST NOT: a second `create_async_engine`.

Same idea for Redis and the vendor HTTP client: **one** client/pool per process, opened in lifespan / runner, closed on shutdown (08). MUST NOT: `httpx.AsyncClient()` per request.

When the pool is exhausted: fail with `ServiceUnavailableError` (05, 503). MUST NOT: wait without a timeout. MUST NOT: open a one-off engine to "get around" the pool.

---

## Timeouts

Every outbound wait has a timeout from `config/`. No timeout = the process is the resource.

MUST:

- Postgres `statement_timeout` on the engine's `connect_args` — the exact wiring is in 06. Migration timeouts stay in 07 and are not a substitute.
- Redis command timeout
- Vendor HTTP: connect + read/total
- Job: a per-`job_type` wall clock; kill/fail → retry or DLQ (11), not a stuck greenlet
- Proxy/platform request timeout **shorter** than "wait forever"; the app still times out its own awaits

Timeouts and vendor 5xx / 429 → `ServiceUnavailableError` / retry policy (11). MUST NOT: catch and ignore. MUST NOT: a 30s Stripe call on the request path — that is a job (Decide 2).

---

## Queries and indexes

The cheap path is one SQL that answers the question.

MUST: repository method named after the question (06). MUST: `list_by_ids` instead of a loop. MUST: relationships loaded in that query (`selectinload`) or `lazy="raise"`.

MUST: an index that matches the live `WHERE` + `ORDER BY` (plus `id` tie-break for cursors, 12). Add it in the same change as the query if the table is not tiny. Live table: `CONCURRENTLY` (07).

MUST NOT: an index on every column. MUST NOT: `COUNT(*)` for `total` unless the client needs that number (12). MUST NOT: `SELECT *` of a wide row when the list DTO needs three columns — load only those columns. If they come from one table, that is a method on its repository; if the screen genuinely spans two tables, that is a **read repository** returning a DTO (06), never a SQL string in the service.

---

## Do not hold a connection

Session scope is the request or the job (06). While it is open, one pool slot is taken.

MUST: `commit()` then `publish` then return (11). MUST NOT: call mail/S3/LLM/HTTP inside the same session as the write if that wait is long — commit, close, then the job opens a **new** session for the next use case.

Streaming a file: `infra/storage/` streams; the module does not `read()` the whole object into RAM. MUST NOT: BYTEA for user files (15). MUST NOT: hold `get_session` during the stream.

---

## Cache (speed, not truth)

Allowed only when Decide 4 is yes. Reconstructible (13). TTL always. On write, the **owning module** deletes or versions the key.

MUST NOT: Redis as the order. MUST NOT: cache an ORM. MUST NOT: a global `lru_cache` on a function that closes over a session or user. `get_settings` in 03 is the exception — frozen config, no request state.

---

## CPU, memory, logs

CPU: stays off the API event loop when it is more than a few milliseconds of tight compute. Job or [Extra 04](extra/04-packages.md) from the job.

Memory: page, stream, cap upload. MUST NOT: build a giant `list` of ORM to serialize.

Logs: QueueHandler so the request does not wait on disk (04). MUST NOT: log bodies, query results, or file bytes. Slow-request / pool-wait belongs in `http/middleware/metrics.py` when that file exists (10) — count and latency by **status class**, not `order_cancelled_total`.

---

## Health vs ready

`GET /health` (10) — liveness. No DB, no Redis. The process is up.

Readiness (platform probe) — may check "can I get a connection **with a short timeout**". MUST NOT: replace `/health` with a probe that fails because Redis is briefly slow (that flaps pods and makes the outage worse). MUST NOT: a module service on either probe.

---

## Done

- [ ] No new cache, replica, or [Extra 02](extra/02-microservices.md) split until Decide 1–3 are false
- [ ] Caps live in `config/`; pool × process_count fits Postgres `max_connections`
- [ ] Worker concurrency ≤ that process's DB pool
- [ ] Every outbound await has a timeout
- [ ] Session is not held across vendor I/O
- [ ] Lists are cursor + `LIMIT`; no N+1; index matches the query
- [ ] Cache still reconstructible (13); no ORM in Redis
- [ ] Unbounded `create_task` / `list_all` / sync I/O in `async def` is absent
