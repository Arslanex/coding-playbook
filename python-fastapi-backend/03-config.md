# 03 · Config

WHEN: reading an env var, adding a limit / timeout / pool size, a secret, or a feature flag.
LOAD: this file only.
RELATED: 02 (where `config/` sits) · 15 (secret handling) · 16 (which numbers must be capped) — open only if the task is also that topic.
SCOPE: `src/config/`. Every other file in this playbook says "from `config/`". This file says what that means.
ADAPTED (GİRVAK, vidinsight-blog-service): env names use a **single** underscore — `SECURITY_JWT_SECRET`, not `SECURITY__JWT_SECRET` — because the double one reads as a typo to everyone who edits the deploy file. `env_nested_delimiter` cannot express that (a single `_` splits on *every* underscore, so `SECURITY_JWT_ACCESS_TTL_SECONDS` would parse as five levels), so each group is its own `BaseSettings` with an `env_prefix`. `settings.security.jwt_secret` is unchanged in code. The cost and its replacement are in *Shape* below.

One typed settings object per process. Built once at startup. Read everywhere by injection, never by `os.getenv`.

---

## The rule the rest of the playbook depends on

MUST: every environment value, operational limit, timeout, pool size, and secret **name** is a field on `Settings`.
MUST NOT: `os.getenv` / `os.environ` outside `config/`. Not in `modules/`, not in `infra/`, not in `http/`, not in `workers/`, not in `main.py`.
MUST NOT: a magic number in `session.py`, a repository, a job file, or a middleware (16).

If a number has no home, that is the trigger to add a field here — not to inline it.

---

## Files

```
src/config/
  __init__.py      # exports get_settings (and Settings for type hints)
  settings.py      # class Settings + nested groups
```

Day one that is enough. Split (01) only when one of 01's triggers fires — in practice, when the groups start having separate reasons to change — then one file per group (`database.py`, `security.py`) and `settings.py` composes them. MUST NOT: split on length; a settings file is long because the product has settings. MUST NOT: `config/base.py` + `config/dev.py` + `config/prod.py` as three classes. One class; the **environment** supplies different values.

MUST NOT: `config/constants.py` for business numbers ("an order may hold 50 items"). That is a product rule — `modules/` (09). `config/` holds operational numbers, not policy.

---

## Shape

Pydantic Settings (`pydantic-settings`), one class, `frozen=True`.

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_nested_delimiter="__",
        extra="forbid",
        frozen=True,
    )

    environment: Literal["local", "ci", "staging", "production"]
    log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"

    database: DatabaseSettings
    security: SecuritySettings
    limits: LimitSettings
```

MUST: `extra="forbid"` — a typo in a deploy variable fails the boot instead of silently using a default.

ADAPTED (GİRVAK): with `env_prefix` groups, `extra="forbid"` cannot sit on the outer model — every `SECURITY_*` key belongs to a group that reads it itself and looks extra to the parent. The protection is not dropped: a `model_validator` scans `.env` and refuses to start on any key that is neither a setting nor a documented exception (`SEED_*`, `SYNC_*`, `DATABASE_URL`, `FORWARDED_ALLOW_IPS`, `RUN_DB_TESTS`). Checking only prefixed keys was tried first and passed `SECURTIY_JWT_SECRET`, where the misspelling is in the prefix — which is the mistake most worth catching. Only `.env` is scanned, never the whole environment: a container carries plenty that is not ours. Pinned by `tests/config/test_settings_env_names.py`.
MUST: `frozen=True` — nothing mutates settings at runtime. A value that must change while the process runs is a feature flag store, not this object.
MUST: every field typed. `Literal` for a closed set, `int` for a count, `timedelta` / `float` seconds for a duration — pick one duration style and keep it. MUST NOT: `str` for a number.

Group into nested models once the flat list passes ~15 fields:

```python
class DatabaseSettings(BaseModel):
    dsn: PostgresDsn
    pool_size: int = 10
    max_overflow: int = 5
    statement_timeout_ms: int = 5_000
```

Env name = the path, upper-cased, joined by the delimiter: `DATABASE__POOL_SIZE`, `SECURITY__JWT_SECRET`.

ADAPTED (GİRVAK): one underscore — `DATABASE_POOL_SIZE`, `SECURITY_JWT_SECRET`. Each group is a `BaseSettings` with `model_config = SettingsConfigDict(env_file=".env", extra="ignore", env_prefix="SECURITY_")`, and `Settings` composes them with `Field(default_factory=...)`. Same object path in code, one underscore in the file.

---

## Defaults

Stop at the first yes.

1. Wrong value causes data loss, a security hole, or a bill (DSN, JWT secret, CORS origins, environment)?
   → **no default**. Required field. The process must not boot without it.
2. Safe operational number that most environments keep (pool size, list `limit` cap, timeout)?
   → default here, override per environment.
3. Value differs per environment but a wrong one is only slower (log level, prefetch)?
   → default = the safest production value, not the convenient local one.

MUST NOT [critical]: `jwt_secret: str = "changeme"`. A default secret ships to production exactly once.
MUST NOT: `debug: bool = True`.

---

## Secrets

MUST: the settings field holds the **value at runtime**, but the value comes from the platform's secret store (env injected by the orchestrator, or a fetch in the settings loader). `.env` is a local-development convenience.
MUST: `.env` in `.gitignore`. `.env.example` is committed with **names and empty values** only.
MUST: `SecretStr` for secrets so a stray `repr(settings)` does not print them.
MUST NOT [critical]: a secret in the repo, in a test fixture copied from production, in CI logs, or in a frontend bundle (15).
MUST NOT: log the settings object at startup. Log the **environment name and the config version**, never the values (04).

---

## Building it

One instance per process, built at startup, injected downstream.

```python
@lru_cache(maxsize=1)
def get_settings() -> Settings:
    return Settings()
```

MUST: `main.py` lifespan and `workers/runner.py` call `get_settings()` **first**, before pools and before `configure_logging()` (04) — a bad config must fail before anything opens a socket.
MUST: validation failure at startup is fatal. Print which field failed, exit non-zero. MUST NOT: catch it and boot with defaults.
MUST: pass what a collaborator needs, not the whole object, when the callee is a helper (`compute_totals(items, tax_rate)`), so the helper stays testable without an environment (09).
MUST NOT: a module-level `settings = Settings()` at import time in a random file — import order then decides whether the app boots.
MUST NOT: `Depends(get_settings)` as the only way to read config in a service. The service takes what it needs through its constructor or the call.

---

## What must live here (checklist for 16)

These are named across the playbook as "from `config/`". If one is missing, that is the defect:

- DB DSN, `pool_size`, `max_overflow`, `statement_timeout` (06, 16)
- Redis URL, socket timeout, pool (08, 16)
- Queue URL, prefetch, per-`job_type` retry attempts and backoff (11)
- Worker concurrency semaphore, per-`job_type` wall clock (11, 16)
- Vendor HTTP: base URL, connect timeout, total timeout, max connections (08, 16)
- JWT signing secret/key, access TTL, refresh TTL (13)
- CORS origins, `/docs` on/off, rate-limit windows and caps (10, 15)
- List `limit` default and cap (12), upload max bytes (15), queue payload max size (11)
- Log level, rotating-file settings if the process owns a disk (04)

---

## Tests

MUST: tests build `Settings` from explicit values or a fixture `.env.test`, not from the developer's shell.
MUST: fake secrets that are obviously fake (`test-jwt-secret`) (14).
MUST: `get_settings.cache_clear()` in a fixture if a test changes the environment. MUST NOT: leave a mutated cache for the next test.
MUST NOT: copy production values into `conftest.py` (14, 15).

---

## Done

- [ ] No `os.getenv` outside `config/`; no magic number in infra / jobs / middleware
- [ ] One frozen `Settings`, `extra="forbid"`, every field typed
- [ ] Secrets are `SecretStr`, absent from git, never logged
- [ ] Required-and-dangerous fields have no default
- [ ] Built once at startup; bad config fails the boot loudly
- [ ] Business policy stayed in `modules/`, not in `config/`
