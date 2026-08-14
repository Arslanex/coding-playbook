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

## File: when to split

Do not split because a topic exists. Split because the file can no longer be edited safely.

SHOULD: keep a file under 300 lines.
MUST: split before 500 lines.

MUST split when any of these is true:

- two reasons to change (e.g. HTTP shape and a business rule in the same file)
- the next agent cannot see the public surface without scrolling past helpers
- tests for one behavior require importing unrelated helpers
- a second `*Service` class appeared — that is a new package, or a helper that is not a Service

MUST: after a split, each file still has one capability in its header `Purpose`.
MUST NOT: split by dumping leftovers into `utils.py` / `helpers.py` / `common.py`.
MUST NOT: split by adding a second `*Service` in the same package. Helpers are named after the work (`totals.py`, `render.py`), not `*_service.py`. How a helper is named, what it holds, and how the service is given it: [09-modules.md](09-modules.md).

---

## File-header docstring

MUST: first bytes of every `.py` file. Nothing above it.
Audience: the next agent choosing a file. After this block they know edit-here vs open-another.

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
`Layer` — one word: `Router` · `Schema` · `Service` · `Repository` · `Model` · `Worker` · `Shared` · `Test`.
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
MUST: typed values across function boundaries (schema, DTO, dataclass, ORM). No bare `dict` as a contract.
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
MUST NOT: mutate a frozen/in-DTO object in place — return a new one.

---

## Done

Stop if any item is false.

- [ ] Each new/changed function has one job; no "and" in the name
- [ ] File still under the split thresholds, or already split by capability
- [ ] File-header has real `Called by` and `Calls`
- [ ] Public functions have `Args` / `Returns` / `Raises`
- [ ] No `#` that restates the next line
- [ ] No user sentence that exists only as a comment
