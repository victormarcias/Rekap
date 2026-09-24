# Testing (pytest)

Concrete implementation with `pytest`, `httpx.AsyncClient`, and mocking external services. The underlying concepts (test pyramid, test doubles, why mock, isolation) are in [Testing — General Concepts](../../system-design/testing.md).

## 1. `pytest` fixtures and `conftest.py`

A fixture is a reusable setup/teardown function. `conftest.py` holds the fixtures shared across the whole project — pytest discovers them on its own, with no test having to import them. Everything after `yield` runs as teardown, always, even if the test fails.

```python
# conftest.py
import pytest

@pytest.fixture
def sample_user():
    return {"email": "ana@mail.com", "password": "secret123"}

@pytest.fixture
def db_session():
    session = SessionLocal()
    yield session       # the test uses the session here
    session.close()      # teardown: always runs, whether the test passed or failed
```

Any test in any file can request `sample_user` or `db_session` as a function parameter — pytest resolves them automatically.

## 2. Transactional rollback in async SQLAlchemy

Applying the [Testing — General Concepts](../../system-design/testing.md) pattern with a real fixture: a transaction opens on the connection, gets used to create the session the test sees, and gets rolled back at the end.

```python
@pytest.fixture
async def db_session():
    async with engine.connect() as conn:
        trans = await conn.begin()
        session = AsyncSession(bind=conn)
        yield session          # the test does its inserts/updates here
        await trans.rollback()  # everything gets reverted — the DB ends up exactly as it was
```

Every test starts from a clean state with no truncating tables or recreating the database, and runs fast because it never actually commits anything to disk.

## 3. `httpx.AsyncClient` for testing async routes

`TestClient` (seen in [Microservice Endpoints](microservice-endpoints.md)) works for both sync and async routes because it manages its own event loop internally, but when the whole app uses async fixtures (like the DB session from point 2), `AsyncClient` fits better: it's awaitable and integrates directly with `pytest-asyncio` with no nested loops.

```python
import pytest
from httpx import AsyncClient, ASGITransport

@pytest.mark.asyncio
async def test_get_order():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        response = await client.get("/orders/1")
    assert response.status_code == 200
```

## 4. Mocking external services

`pytest-mock` for your own functions (like sending email); `moto` for simulating entire AWS services (like S3) without hitting the real cloud.

```python
# ✅ mocking sending mail: the test doesn't send any real email
def test_register_sends_welcome_email(mocker):
    mock_send = mocker.patch("app.services.email.send_welcome_email")
    client.post("/register", json={"email": "ana@mail.com", "password": "secret123"})
    mock_send.assert_called_once_with("ana@mail.com")

# ✅ moto: simulates S3 in memory, without hitting real AWS or spending anything
from moto import mock_aws
import boto3

@mock_aws
def test_upload_to_s3():
    s3 = boto3.client("s3", region_name="us-east-1")
    s3.create_bucket(Bucket="my-test-bucket")
    upload_file(s3, "my-test-bucket", "photo.jpg", b"content")
    objects = s3.list_objects(Bucket="my-test-bucket")
    assert objects["Contents"][0]["Key"] == "photo.jpg"
```

## 5. Testing auth-protected routes

Reusing the same helper `/login` uses in production (see [Authentication in FastAPI](authentication.md)) to generate a valid token inside a fixture, instead of actually logging in on every test.

```python
@pytest.fixture
def auth_headers():
    token = create_access_token(user_id=1)  # same helper as /login in production
    return {"Authorization": f"Bearer {token}"}

def test_get_me(client, auth_headers):
    response = client.get("/me", headers=auth_headers)
    assert response.status_code == 200
```

## 6. Testing file uploads

```python
def test_upload_avatar(client, auth_headers):
    file_content = b"fake-image-bytes"
    response = client.post(
        "/users/me/avatar",
        headers=auth_headers,
        files={"file": ("avatar.jpg", file_content, "image/jpeg")},
    )
    assert response.status_code == 200
```

## 7. Testing background tasks

In production, a FastAPI `BackgroundTask` runs after the response is sent. In tests, since `TestClient`/`AsyncClient` run everything in the same process, the task has already finished by the time the assertion runs — no need to wait or poll, just mock the function and verify it was called.

```python
def test_create_order_triggers_notification(client, auth_headers, mocker):
    mock_notify = mocker.patch("app.services.notifications.notify_warehouse")
    response = client.post("/orders", headers=auth_headers, json={"items": ["sku-1"]})
    assert response.status_code == 201
    mock_notify.assert_called_once()  # the background task already ran by the time we get here
```

## 8. Testing ownership/authorization checks

The classic case: an authenticated user tries to touch a resource that isn't theirs — the correct response is `403`, not `401` (see [Authentication vs Authorization](../../backend/authentication.md)).

```python
def test_cannot_delete_other_users_order(client, auth_headers_user_a, order_owned_by_user_b):
    response = client.delete(f"/orders/{order_owned_by_user_b.id}", headers=auth_headers_user_a)
    assert response.status_code == 403  # authenticated, but not authorized for this resource
```

## 9. Setup/teardown hooks in pytest (fixture `scope`)

pytest doesn't have `beforeEach`/`afterAll` as separate functions (see the general concept in [Testing — General Concepts](../../system-design/testing.md#6-setupteardown-hooks--beforeeach-aftereach-beforeall-afterall)) — it solves the same thing with a fixture's `scope` parameter: `scope="function"` (default) is equivalent to `beforeEach`/`afterEach`; `scope="session"` (or `"module"`, to share only within one file) is equivalent to `beforeAll`/`afterAll`.

```python
@pytest.fixture(scope="function")  # default — runs before/after EVERY test, like beforeEach/afterEach
def db_session():
    session = SessionLocal()
    yield session       # equivalent to beforeEach
    session.close()      # equivalent to afterEach

@pytest.fixture(scope="session")  # runs ONCE for the whole run, like beforeAll/afterAll
def test_db_connection():
    conn = connect_test_db()
    yield conn
    conn.close()
```

| JS (Jest/Mocha) | pytest |
|---|---|
| `beforeEach` / `afterEach` | fixture with `scope="function"` (default) |
| `beforeAll` / `afterAll` | fixture with `scope="session"` (or `"module"`) |

Same care as in JS: a `scope="session"` fixture that should actually reset per test breaks isolation — see [Test isolation](../../system-design/testing.md#4-test-isolation).

---
Related: [Testing — General Concepts](../../system-design/testing.md), [Microservice Endpoints](microservice-endpoints.md), [Authentication in FastAPI](authentication.md), [Sync vs Async in FastAPI](sync-vs-async.md).
