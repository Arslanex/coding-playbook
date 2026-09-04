# Extra 04 · Your library / package

WHEN: you wrote (or will write) a **library** this backend should use — execution engine, agent framework, ML trainer/inferencer, PDF kit, anything that is not a product noun and not a vendor SDK.
LOAD: this file **and** [02](../02-file-structure.md), [08](../08-infra.md), [09](../09-modules.md). Not instead of them.
SCOPE: **where it sits in the tree** and **how `backend/src` imports it**. Not the internals of that library.

This Extra is not "the execution Extra." Execution, agents, and ML are **examples** of packages. The rule is the same for all of them.

---

## Decide: package vs module vs infra

Stop at the first yes.

1. Deleting a **vendor** (SageMaker, OpenAI, Docker) would delete the file, and no product sentence remains?
   → `infra/<capability>/` client (08). Wrap the vendor. Do not invent a company library for one `boto3` call.
2. The code **is** a product ability ("user starts a training job", "user starts a run")?
   → `modules/<capability>/` (09) + worker (11). The library is **called from** the service, it is not the module.
3. Reusable engine with **no** `orders` / `auth` in its API (execute bytes, train on a matrix, agent loop kernel)?
   → a real package (below). `backend` depends on it.
4. Used in one module, 40 lines, no second caller?
   → helper file in that module (09). MUST NOT: a package "for cleanliness."

MUST NOT: `src/core/ml.py`, `src/utils/execution`. MUST NOT: put the library inside `modules/` so HTTP schemas leak into it.

---

## Where in the directory

**Same repo as this backend** (default):

```
/
├── backend/                    # playbook tree (02)
│   └── src/modules/…           # imports the package
├── packages/
│   └── <name>/                 # execution | agentkit | trainer | …
│       ├── pyproject.toml
│       ├── src/<name>/
│       └── tests/
└── deploy/
```

`<name>` is the kernel's job: `execution`, `agentkit`, `trainer`. MUST NOT: `common`, `core`, `lib`.

**Own repo**: same package layout; backend adds a versioned dependency. [Extra 02](02-microservices.md) if that kernel is also a **service**.

Install: workspace path or pinned version in `backend` pyproject. MUST NOT: `sys.path.append`.

The package has its own tests (14 mirror inside `packages/<name>/tests`). 01 file-headers. MUST NOT: the package import `backend.src.modules`.

---

## How it is included in this backend

The app **owns meaning**. The package **owns the engine**.

```
modules/<capability>/service.py
  → builds a command (DTO / dataclass: ids, paths, limits from config)
  → calls package API  (pure or with injected ports)
  → or enqueue worker (11) that calls the package

workers/jobs/<kind>.py
  → session, *Service, package.run(...)
  → persist results via the service (commit)

infra/<vendor>/                 # only if the package needs a system we don't own
  → sandbox runtime, GPU queue, S3 for artifacts
```

Injection: the worker/service passes **ports** (storage put/get, "run this tool callable", clock). The package MUST NOT: open `infra/db/session.py` or `OrderRepository`.

HTTP: never block on a long train/execute. `202` + id (12). Artifact: `infra/storage/` key chosen by the **module**.

Examples — same wiring, different package name:

- **Execution system** (speed up / isolate jobs): `packages/execution` — timeouts, sandbox handle, step loop. Module: `modules/runs/` if users see runs; else orders/mail just call `execution.run` from a job.
- **Agent framework**: `packages/agentkit` — model loop, tool dispatch. Tools injected as `*Service` methods ([Extra 03](03-agent-teams.md)). Vendor LLM: `infra/llm/` or inside the package **adapter** if the package owns "complete(prompt)".
- **Machine learning**: `packages/trainer` / `packages/infer`. Train job in workers; weights in `infra/storage/`; live infer either in-process if cheap or [Extra 02](02-microservices.md) if GPU is its own deployable. Labels/features that **mean** "order fraud" stay in `modules/…`; tensors stay in the package.

---

## What the package must not do

MUST NOT: know `/v1`, JWT, `tenant_id` as a required kernel field (caller passes an opaque `context` dict if needed).
MUST NOT: `eval` user code unless [Extra 03](03-agent-teams.md)+15 sandbox rules are met **in infra**, not as a hidden default.
MUST NOT: log secrets (04). MUST NOT: be the source of truth for product rows (06).

---

## Done

- [ ] Decide first-yes: infra vendor vs module vs `packages/<name>` vs helper
- [ ] Package sits at `packages/<name>/` (or its own repo), not under `core/` / `utils/`
- [ ] Backend service/worker calls it; package does not import modules or repos
- [ ] Long work is a job + 202; artifacts in storage; meaning in the module
