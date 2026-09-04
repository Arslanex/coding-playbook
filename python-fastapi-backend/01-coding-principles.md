# 01 · Coding principles

WHEN: every Python edit, before the first line.
LOAD: this file only. Placement and folder maps live elsewhere — do not infer layout from this document.
RELATED: 02 (where the file goes) · 09 (how a helper is named and given) — open only if the task is also that topic.
SCOPE: all Python the agent writes or changes.

---

## Function: one job

MUST: a function does one thing. Its name is that thing.

MUST NOT: a name that needs "and" (`create_and_notify`, `load_and_validate`). Split.

MUST: extract a step when it has its own rule, error, or reason to change — even if it is three lines.

SHOULD: body < 30 lines. Above that, the function is doing more than one job.

---

## Function: signature

A call you cannot read without opening the definition is a call the next agent will get wrong.

MUST: at most **three** parameters.

Not counted — these are not choices you made:

- `self` / `cls`
- anything supplied by `Depends(...)`: session, current user, settings, an injected repository
- a router's **path parameters** and its **single** request-body schema. That signature is the HTTP contract and FastAPI reads it (10, 12). Four or more *query* parameters still collapse into one params model.
- `__init__` receiving injected dependencies. SHOULD: no more than four of those — a service that needs five collaborators owns too much (09).

WHEN: you reach four. Stop at the first that applies:

1. **They do not travel together** → the function has more than one job. Split it, do not package it.
2. **They travel together and describe one thing** → pass a DTO (below).

MUST NOT: reach for a DTO to make an oversized function legal. A long parameter list is usually a cohesion problem wearing a packaging costume — check 1 before 2.

MUST NOT: a boolean flag parameter (`force=`, `dry_run=`, `send_email=`). A flag means the body has an `if` that splits it into two behaviours; those are two functions with names. Exception: a flag the framework or a library requires.

SHOULD: keyword arguments at the call site for everything except the primary subject. `cancel(order_id, reason=reason, actor=user.id)` survives a signature change; three bare positionals do not.

---

## What crosses a function boundary

MUST: a typed value. Never a bare `dict` as a contract — a `dict` moves the contract into the caller's head and no tool can check it.

Two shapes, and they are not interchangeable:

**Pydantic schema** — the **HTTP boundary only**. Request and response bodies in the module's `schemas.py` (09, 12).
MUST NOT: pass a request or response schema between services, into a repository, or into a worker. The wire shape then becomes the domain shape, and the next API version change reaches into code that has nothing to do with HTTP.

**DTO** — a frozen dataclass, for everything internal: a value that crosses a package boundary, or must outlive the session it was read in (06, 09).
MUST: frozen. MUST NOT: mutate one in place — return a new one.

MUST NOT: return a tuple of unrelated values (`return order, total, warnings`). The caller unpacks by position and a reordering breaks it silently. Return a DTO.
MUST NOT: name a DTO `Data`, `Params`, `Info`, or `Context`. Name the thing it is — `CancellationRequest`, `OrderTotals`.

WHERE it lives: the module that owns the meaning; a read-repository's DTO stays next to that repository (06, 09). MUST NOT: a `core/dto.py` collection.

---

## File: when to split

Do not split because a topic exists. Do not split because a file is long. Split
because the file can no longer be edited safely.

**There is no line count here, deliberately.** One was tried and it failed in
both directions. It fired falsely: this page requires a docstring with `Args`,
`Returns` and `Raises` on every public function, so a service crossed the
threshold *because* it was documented — 691 lines of which 303 were code — and
the split it demanded would have cut a coherent file in half to satisfy
arithmetic. And it stayed silent when it mattered: a file under any threshold
can still hold two reasons to change, and a number tells you it is fine.

A count is easy to check, which is exactly why it gets checked instead of the
thing it stands for. The triggers below are the rule. Read them.

MUST split when any of these is true:

- two reasons to change (e.g. HTTP shape and a business rule in the same file)
- the next agent cannot see the public surface without scrolling past helpers
- tests for one behavior require importing unrelated helpers
- a second `*Service` class appeared — that is a new package, or a helper that is not a Service

If none of them is true, the file is the right size whatever its length.

MUST: after a split, each file still has one capability in its header `Purpose`.
MUST NOT: split by dumping leftovers into `utils.py` / `helpers.py` / `common.py`.
MUST NOT: split by adding a second `*Service` in the same package. Helpers are named after the work (`totals.py`, `render.py`), not `*_service.py`. How a helper is named, what it holds, and how the service is given it: [09-modules.md](09-modules.md).

---

## File-header docstring

MUST: first bytes of every `.py` file. Nothing above it.
Audience: the next agent choosing a file. After this block they know edit-here vs open-another.

ADAPTED (GİRVAK, vidinsight-blog-service): the header is written as **prose**, not this field block — `"""Layer: <one word>. <what this file is for>.` then a `Called by:` / `Calls:` line, then a paragraph saying why the file exists at all. It carries four of the six fields below (`Layer`, `Purpose`, `Called by`, `Calls`) and adds the reason, which the template does not ask for and which is the thing a later agent actually needs before deciding to edit here. `Module:` is dropped: it restates the path of the file the reader has already opened, and 88 of them is 88 lines that go stale on a rename. `Dependencies:` is kept where a file takes injected types. The audience argument above is unchanged and is the reason the prose form must still open with `Layer:` and still name concrete files. Pinned by `tests/test_docstrings.py::test_every_file_has_a_header`.

```python
"""
Module: orders/service.py
Layer: Service
Purpose: Create, cancel, and list orders for the calling user.
         Totals live here. Payment capture does not.

Dependencies:
    - OrderRepository: row access
    - BillingService: capture on confirm

Called by: orders/router.py, jobs/issue_invoice.py
Calls: infra/db/repositories/order.py, billing/service.py
"""
```

`Module` — file identity (`orders/service.py`).
`Layer` — one word: `Router` · `Schema` · `Service` · `Repository` · `Model` · `Worker` · `Core` · `Test`.
`Purpose` — allowed work. Second sentence = what this file must not do, if a later agent would guess wrong.
`Dependencies` — injected types.
`Called by` / `Calls` — concrete files.

MUST: English.
MUST NOT: function inventory in `Purpose`.
MUST NOT: vague `Called by: the API` / `Calls: the database`.
MUST (tests): same block. `Layer: Test`. `Called by: pytest`. `Calls:` module under test. Path and what to assert: [14-testing.md](14-testing.md).

---

## Function docstring

Audience: the next agent calling the function. Not the product user.

MUST: every public function (`def` not starting with `_`).

```python
async def cancel(self, order_id: uuid.UUID, user_id: uuid.UUID) -> Order:
    """Cancel an order the caller owns. Paid orders are not cancelled here.

    Args:
        order_id: Order to cancel.
        user_id: Caller. The order must belong to this user.

    Returns:
        The order in cancelled status.

    Raises:
        OrderNotFoundError: Missing, or not owned by user_id.
            Same error on purpose — a 403 would confirm the id exists.
        OrderNotCancellableError: Already paid or already cancelled.
    """
```

MUST: `Args`, `Returns`, `Raises`.
MUST: write the invariant a later agent would break.
SHOULD: `_private` — one line, or omit if the name is the line.
MUST NOT: duplicate a callee's docstring on the caller. Caller: what this entrypoint adds (status, permission). Callee: business `Raises`.

---

## Comments

Two audiences. Never mixed.

Agent (source):

- File-header docstring — always
- Public function docstring — always
- `#` inline — only if the next agent would delete or reorder the wrong thing (invariant, required order, non-obvious constraint)

```python
# Same error for missing and not-owned — do not split into 404/403.
# Authenticate before opening the session.
# Fetch limit+1 so the caller can detect truncation.
```

MUST NOT: `# get the order` / `# return the response`
MUST NOT: commented-out code — delete it
MUST NOT: changelog comments

User (product) — never reads `#` or docstrings.

- Error `message` → exception constructor
- Payload copy → return value / schema field
- Operator text → logger call + structured fields

MUST: docstrings English (agents).
MUST: user-visible `message` in the product language.
MUST NOT: park UI copy in a comment.

```python
# WRONG
# User: "Sipariş iptal edilemez"

# RIGHT
class OrderNotCancellableError(ConflictError):
    error_code = "ORDER_NOT_CANCELLABLE"

    def __init__(self) -> None:
        super().__init__("Bu sipariş iptal edilemez")
```

---

## Readable, changeable code

MUST: names are verbs for functions (`cancel`, `list_for_user`), predicates for bools (`is_cancellable`).
MUST NOT: new types named `data`, `info`, `manager`, `helper`, `util`.
MUST: `Decimal` for money and any exact quantity. MUST NOT: `float` — `0.1 + 0.2` is a support ticket. Column type: 06.
MUST: timezone-aware `datetime`, UTC. MUST NOT: `datetime.utcnow()` or `datetime.now()` without a timezone; a naive value never crosses a function boundary. Inject a clock where a test must freeze time (14).
MUST: `Enum` for a closed set of values, not string literals compared by hand.
MUST: I/O (DB, network, disk, queue) is `async def`.
MUST NOT: blocking I/O inside `async def`.
SHOULD: independent awaits via `asyncio.gather`.
MUST: raise typed exceptions. MUST NOT: return `None` to mean "not found" when the caller must handle absence — raise or return an explicit type.
MUST NOT: catch-all `except Exception` unless it re-raises or translates to a typed error.
MUST: import order — stdlib, third party, local. Blank line between groups.
MUST NOT: unused imports, unused locals.

---

## Done

Stop if any item is false.

- [ ] Each new/changed function has one job; no "and" in the name
- [ ] No signature over three real parameters; no boolean flag parameter
- [ ] Values crossing a boundary are typed — DTO internally, Pydantic schema only at HTTP; no bare `dict`, no unrelated tuple
- [ ] File still under the split thresholds, or already split by capability
- [ ] File-header has real `Called by` and `Calls`
- [ ] Public functions have `Args` / `Returns` / `Raises`
- [ ] No `#` that restates the next line
- [ ] No user sentence that exists only as a comment
