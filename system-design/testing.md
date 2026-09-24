# Testing — General Concepts

Testing fundamentals that apply in any language. The concrete implementation with pytest/FastAPI is in [Testing in FastAPI](../stacks/fastapi/testing.md).

## 1. Test pyramid

Tests are organized into layers based on how much they cover and how much they cost: **unit tests** (many, fast, test an isolated function/class), **integration tests** (fewer, test several components together — e.g. that an endpoint actually persists to a test DB), and **e2e tests** (few, test the whole system through the real interface, the slowest and most fragile).

```
        /\
       /e2e\        few, slow, lots of confidence, break often for reasons unrelated to the bug
      /------\
     /integr. \     some, test that the pieces fit together
    /----------\
   /   unit     \   many, fast, isolated — the base of the pyramid
  /--------------\
```

Inverting the pyramid (few unit tests, many e2e — the "ice cream cone" anti-pattern) gives you a slow, flaky test suite: every bug takes minutes to confirm instead of milliseconds, and a network or browser hiccup breaks tests that have nothing to do with the real bug.

## 2. Test doubles — Mock vs Stub vs Fake vs Spy

Used as synonyms, and they aren't — each solves a different problem when replacing a real dependency in a test:

| Type | What it does | Verifies the "how" it was called |
|---|---|---|
| **Stub** | Returns a fixed response, no real logic | ❌ |
| **Mock** | Like a stub, but also records and lets you verify the calls it received | ✅ |
| **Fake** | A real but simplified implementation (e.g. an in-memory DB instead of Postgres) | ❌ |
| **Spy** | Wraps a real object, delegates the real call and also records it | ✅ |

```python
# Stub: returns a fixed value, doesn't care how or how many times it was called
class StubPaymentGateway:
    def charge(self, amount, card):
        return {"status": "succeeded"}

# Mock: besides the return value, lets you verify the "how" —
# this is what distinguishes a mock from a plain stub
mock_gateway = Mock()
mock_gateway.charge.return_value = {"status": "succeeded"}
checkout(mock_gateway, cart)
mock_gateway.charge.assert_called_once_with(100, "4242-...")  # ✅ verifies the call, not just the result
```

## 3. Why mock external services

A test that hits a real external service (email, payment gateway, S3) is **slow** (real network latency), **non-deterministic** (the service can be down, rate-limited, or return something different each time), and can have **real side effects** (actually sending an email, actually charging a card, uploading a real file to a bucket that costs money). Mocking it isolates the test from the outside world: it runs in milliseconds, always the same way, with no real credentials needed in CI.

## 4. Test isolation

A test shouldn't depend on the order it runs in, or leave state that affects the next one. The clearest sign something's wrong: tests pass when run individually but fail when run all together (or in a different order) — that means there's shared state (a global variable, a row left in the DB) that one test is inheriting from another without declaring it as a dependency.

## 5. Transactional rollback pattern for DB tests

Each test runs inside a transaction that opens before it and gets reverted (`ROLLBACK`) at the end, instead of committed. The test can freely insert, update, and delete — none of it stays persisted, so the next test always starts from the same clean state, without having to truncate tables or recreate the database between tests.

```python
# conceptual pattern, framework-independent
def run_test_in_transaction(test_fn):
    transaction = db.begin()
    try:
        test_fn()
    finally:
        transaction.rollback()  # nothing the test did stays persisted
```

This is what lets a test suite with hundreds of integration tests against a real DB keep running in seconds instead of minutes.

## 6. Setup/teardown hooks — `beforeEach`, `afterEach`, `beforeAll`, `afterAll`

Most testing frameworks offer four distinct moments for code that runs **around** the tests, not as part of them (these names are understood to be from JS — Jest/Mocha/Vitest —, but the concept repeats in pytest, JUnit, RSpec, etc. with different syntax):

- **`beforeEach`**: runs before **every** test in the block — the typical place to reset state (create a fresh object, clear a test DB).
- **`afterEach`**: runs after **every** test — specific cleanup.
- **`beforeAll`**: runs **once**, before the block's first test — for expensive, shareable setup (spinning up a test server, opening a connection).
- **`afterAll`**: runs **once**, after the last test — final cleanup of whatever was opened at the start.

```python
# conceptual equivalent in Python — the exact syntax varies by framework
def before_all():       # equivalent to beforeAll — once, before all tests in the block
    return connect_test_db()

def after_all(db):       # equivalent to afterAll — once, at the end
    db.close()

def before_each(db):     # equivalent to beforeEach — before EVERY test, key for isolation
    db.clear()

def after_each():        # equivalent to afterEach — after every test
    reset_mocks()
```

**Why "once" vs "every time" matters**: using `beforeAll` for something that should be reset per test breaks [test isolation](#4-test-isolation) from point 4 — if a test mutates state that `beforeAll` only created once, the next test inherits that mutation without declaring it as a dependency. Practical rule: `beforeAll`/`afterAll` for expensive, genuinely shareable work; `beforeEach`/`afterEach` for anything that needs to start clean on every test.

pytest doesn't have these four functions as such — it solves the same thing with a fixture's `scope` parameter (see the full implementation in [Testing in FastAPI](../stacks/fastapi/testing.md#9-setupteardown-hooks-in-pytest-fixture-scope)).

## 7. Brittle tests vs resilient tests

Not a formal dichotomy with its own name like "Mock vs Stub" — it's more a direct consequence of **what** gets tested. A brittle test verifies **implementation details**: it breaks with any internal refactor, even if the external behavior is still correct. A resilient test verifies **observable behavior** (the result, the contract) — it survives refactors because it doesn't care *how* the result was reached, only *that* it's correct.

```python
# ❌ brittle: tests implementation — breaks if the internal call order changes,
# even if the final result (the discount applied) is still correct
def test_apply_discount_calls_repo_in_order(mocker):
    repo = mocker.Mock()
    service = OrderService(repo)
    service.apply_discount(1, 10)
    assert repo.method_calls == [call.find_by_id(1), call.save(ANY)]  # internal detail, not behavior

# ✅ resilient: tests behavior — doesn't care how the result was computed
def test_apply_discount_reduces_total():
    repo = FakeOrderRepository({1: Order(id=1, status="pending", total=100)})
    service = OrderService(repo)
    order = service.apply_discount(1, 10)
    assert order.total == 90  # the observable result, regardless of the internal path
```

It's the maxim of "test behavior, not implementation" (popularized by Kent C. Dodds): if a refactor that doesn't change any observable result breaks a test, that test was coupled to the implementation, not the contract.

## 8. Black box vs white box

- **Black box**: testing external behavior without looking at or knowing the internal implementation — only what goes in and what comes out matters, per the contract/spec. The test could be written without having read a single line of the code.
- **White box**: testing while knowing the internal implementation — cases are designed by looking at the code: every branch of an `if`, every loop, every possible path, aiming for full code coverage.

```python
def calculate_shipping(weight, country):
    if country == "AR":
        return weight * 2 if weight > 10 else 10
    return weight * 5

# ✅ black box: cases come from the contract/spec, without looking at the code —
# "shipping to Argentina costs this, to another country costs that"
def test_shipping_ar_light():
    assert calculate_shipping(5, "AR") == 10

def test_shipping_other_country():
    assert calculate_shipping(5, "US") == 25

# ✅ white box: cases come from looking at the code's branches —
# you have to see the if/else to know weight > 10 in AR is a different path
def test_shipping_ar_heavy_branch():
    assert calculate_shipping(15, "AR") == 30  # covers the branch the black box tests above skipped
```

**Trade-off**: black box is more resilient to refactors (doesn't care how it's built — see point 7) and better reflects the contract the real user cares about. White box finds edge cases the spec didn't explicitly cover (that `weight > 10` branch maybe nobody asked for, but it exists in the code and needs covering) — it's the basis of *code coverage* metrics. In practice, most of a team's tests are black box at the behavior level, with white box used selectively to hunt down uncovered branches.

| | Black Box | White Box |
|---|---|---|
| What it looks at | Only input/output — the spec/contract | The internal code — branches, paths |
| Needs to see the code? | ❌ | ✅ |
| Example case | "Shipping to Argentina costs X" | "I know there's a different `if weight > 10`" |
| Main risk | May miss a rare internal branch the spec doesn't mention | Fragile against refactors that don't change external behavior |
| Who usually writes it | QA, or anyone with the spec in hand | The dev who wrote that function |

## 9. Fixtures

A **fixture** is the known, reproducible state a test starts from — test data, an already-built object, or an already-prepared environment (a connection, a test user). The idea is that a test never depends on "whatever's left over" from a previous run: it always starts from the same explicitly declared starting point.

It's the concrete mechanism behind two things already covered in this file:
- It solves [test isolation](#4-test-isolation) from point 4 — each test gets its own fresh state, instead of inheriting another test's.
- It's a way of implementing the [setup/teardown hooks](#6-setupteardown-hooks--beforeeach-aftereach-beforeall-afterall) from point 6 — in frameworks like pytest, a fixture directly **replaces** those hooks (depending on its `scope`), instead of coexisting as a separate mechanism.

```python
# fixture as data: an already-built test object, ready to use
fixture_user = {"id": 1, "email": "test@mail.com", "role": "admin"}

# fixture as a function that provides that state, with its own setup/teardown
def user_fixture():
    user = create_test_user()
    try:
        yield user
    finally:
        delete_test_user(user.id)
```

Not every framework models it the same way: in pytest it's an **injectable function** (with its own setup/teardown via `yield`); in many JS frameworks it's more common to be **static data** (a JSON file/example object) loaded at the start of the test. The underlying idea — known, reproducible state — is the same in both cases.

See the pytest implementation in [Testing in FastAPI](../stacks/fastapi/testing.md#1-pytest-fixtures-and-conftestpy).

## 10. Running tests in parallel vs serial

Many test runners can split tests across several workers/processes to run in **parallel** and use the machine's cores — in some this is opt-in (has to be explicitly requested), in others it's the default. Forcing the opposite — running everything **sequentially, in a single process**, one after another — is useful in two typical cases:

- **Debugging a flaky test** (a test that sometimes passes and sometimes fails, **with no code changes** — the sign of something non-deterministic involved, not a real bug in the logic): if you suspect broken [test isolation](#4-test-isolation) (two parallel tests stepping on a shared resource — a port, a file, a DB row), running serial isolates the variable: if the test stops failing, it confirms the problem was parallelism/shared state, not the test itself.
- **CI with limited resources**: on a runner with few cores or little memory, spinning up N parallel workers can end up slower (due to contention) than running everything serial, or straight up run out of memory.

```python
# pytest: serial is the default — parallelism is opt-in with the pytest-xdist plugin
pytest              # serial, one test after another
pytest -n auto      # parallel, splits across workers — this is what gets "turned off" to go back to serial
```

Every ecosystem names this differently (in Jest, JS, it's the `--runInBand` flag) but the concept is the same in any language: force a single process to eliminate parallelism as a variable.

**The real root cause**: if running serial "fixes" a test that was failing in parallel, the problem isn't parallelism itself — it's that the test wasn't really isolated, it shared state with another. Forcing serial is a diagnostic/workaround to confirm the suspicion, not the fix: the real fix is guaranteeing every test has its own [fixture](#9-fixtures) with nothing shared with the others.

---
Related: [Testing in FastAPI](../stacks/fastapi/testing.md), [ACID / transactions / isolation levels](../database/acid.md) (transactions and rollback), [Idempotency](quality-attributes.md).
