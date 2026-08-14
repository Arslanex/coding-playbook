# 04 · Logging

WHEN: adding a log line, a logger name, a filter, a formatter, a handler, or startup logging.
LOAD: this file only.
RELATED: 02 (the package lives in `shared/logging/`) · 12 (`request_id` as the client-facing trace id) · 03 (log level, rotation settings) — open only if the task is also that topic.
SCOPE: every process that emits logs (API and workers).

The pipeline is one. Named loggers are labels, not separate stacks.

---

## Pipeline

```
get_logger(name)
  → logger (propagate False)
    → QueueHandler
      → QueueListener (own thread)
        → ContextFilter   (copy contextvars onto the record)
        → RedactFilter    (mask secret-looking keys)
        → JsonFormatter   (serialize only — no masking, no field injection)
        → handlers in parallel:
            StreamHandler(stdout)          MUST
            RotatingFileHandler            only if the process owns a disk
```

Redaction is a **filter**, not a formatter concern. Two reasons: a second handler with a different formatter would otherwise ship unmasked values, and a filter can drop the record entirely (probe noise) while a formatter cannot.

MUST: stdlib `logging`. MUST NOT: a second logging library.
MUST: `configure_logging()` once at process start (`main.py` lifespan, worker runner). Idempotent — a second call must not duplicate handlers.
MUST NOT: a module call `basicConfig`, `addHandler`, `setLevel`, or create its own handler.
MUST: modules only `get_logger(LoggerName.…)` then `.info` / `.warning` / `.error`.

QueueHandler: the request/worker thread must not wait on disk. Stdout is the container sink. Rotating file only if the process owns a disk — beside stdout, never instead, never as the only sink in a pod.

Named loggers share this pipeline. They differ only in `record.name`. MUST NOT: a formatter or queue per name.

---

## Where the package lives

`shared/logging/` — kernel. No product noun.

Holds: setup, logger names, context bind/reset, filters, JSON formatter, queue listener, stdout handler, optional rotating handler.

Does not hold: "order cancelled" messages. Those stay at the call site in `modules/` / `http/` / `workers/` / `infra/`.

---

## Logger name

Pick by where the call site lives, not by the event.

- `api` — `http/` (middleware, deps, error map)
- `system` — `modules/`, `infra/`, `config/`, `shared/`
- `worker` — `workers/`
- `error` — `http/errors/` unhandled 5xx **only**. MUST NOT: workers or module services use this name.
- `audit` — a state change that must be reconstructible (create/cancel/permission/money). Same pipeline. Downstream filters `"logger": "audit"`

MUST: `get_logger(LoggerName.SYSTEM)` (enum). MUST NOT: `logging.getLogger("systm")` — a typo would be a silent unconfigured logger.

A module service uses `system`. It uses `audit` as a second logger only for those reconstructible writes, not for every info line.

---

## Filters and formatter

Order on the listener side, before emit:

1. Context filter — copy contextvars onto the record. Never drops. Fields always present; missing values are `null`, not omitted.
2. Redact filter — mask `extra=` values whose key looks like a secret (list below). Runs on the record, so every handler is covered.
3. Formatter — one-line JSON, stable keys. Serialisation only. MUST NOT: the formatter add fields or mask values.

Context fields (always in the JSON):

- `timestamp` ISO-8601 UTC
- `level`
- `logger` (the name: `api` / `system` / …)
- `message`
- `request_id` — HTTP: middleware binds it. Worker: bind the job id. `null` only if configure ran before any bind.
- `user_id` — `null` until auth has run

MUST NOT: require `tenant_id` unless the product has tenants. If it does, bind it the same way as `user_id` (null until resolved).
MUST NOT: pass `request_id` / `user_id` again in `extra=` — the filter injects them. `extra=` is for the event (`order_id`, `job_id`).
MUST: `extra=` keys that contain `password`, `secret`, `token`, `authorization`, `api_key`, `cookie`, `session`, `otp`, `private_key` are redacted to `***`. Diagnostic keys `error_code` and `status_code` are not redacted.

---

## Call site

MUST NOT: `print()`.
MUST: message in English, event facts in `extra=`. User-facing copy is not a log message (see 01).

Levels:

- `DEBUG` — local diagnosis. MUST NOT: default production level.
- `INFO` — a completed step (request finished, job finished, order created).
- `WARNING` — recovered, but unexpected.
- `ERROR` — failed; MUST `exc_info=True` when an exception is in scope.
- `CRITICAL` — process cannot continue.

MUST NOT: log every health/ready probe at INFO. Drop in a filter or log at DEBUG.

```python
logger = get_logger(LoggerName.SYSTEM)

logger.info("order_created", extra={"order_id": str(order.id)})

logger.error("order_cancel_failed", exc_info=True, extra={"order_id": str(order_id)})

audit = get_logger(LoggerName.AUDIT)
audit.info(
    "order_cancelled",
    extra={
        "resource": "order",
        "resource_id": str(order.id),
        "action": "cancel",
        "previous_state": previous,
        "new_state": "cancelled",
    },
)
```

MUST: bind context at the edge, not at the call site.

- HTTP: `http/middleware/` binds `request_id` at entry, resets in `finally`. Auth fills `user_id` when known.
- Worker: bind `request_id=job_id` (and `user_id` if the payload has one) before `service.…`, reset in `finally`.

---

## Tracing (optional)

`request_id` is the correlation id this playbook requires (12). It already joins HTTP → queue → worker, because the enqueuing service copies it into the job envelope (11). That is enough for support and for log-store queries.

Distributed tracing is **additive**, and only when spans across processes are a real need:

- MUST: OpenTelemetry auto-instrumentation, not a hand-rolled span tree.
- MUST: add `trace_id` / `span_id` to the **context filter** so a log line joins a span. One pipeline, one filter — MUST NOT: a second logging stack for traces.
- MUST NOT: replace `request_id` with `trace_id`. The client echoes `X-Request-ID` (12); a sampled-away trace must not lose the correlation.
- MUST NOT: a tracing SDK in `shared/logging/`. Wiring is `main.py` / `runner.py`; the exporter endpoint is `config/` (03).

Until an exporter is actually deployed: zero files.

---

## Handlers

MUST: both API and worker processes use the same `configure_logging()`.

Stdout handler: `StreamHandler(sys.stdout)`, JSON, UTF-8. Production sink.

Rotating file handler: same formatter. `when` / `maxBytes` + backup count from `config/`. MUST: still behind QueueListener. MUST NOT: a module open `logs/api.log` itself.

If a rotating file is enabled, one file for the process is enough. MUST NOT: five files for five logger names — split in the log store by the `logger` field. Exception: a compliance volume that must keep `audit` alone — then a second rotating handler with a filter that lets only `record.name == "audit"` through. Still on the same listener.

Root logger: level WARNING, same queue/stdout path, so libraries do not print foreign formats. Application loggers: `propagate = False`.

---

## Done

- [ ] No new handler/filter/formatter outside `shared/logging/`
- [ ] Call site uses `get_logger(LoggerName.…)` only
- [ ] `request_id` / `user_id` come from context, not copied into every `extra=`
- [ ] Secrets would be redacted if they appeared in `extra=`
- [ ] Exceptions use `exc_info=True`
- [ ] Queue still sits in front of every handler
- [ ] Production still has stdout
