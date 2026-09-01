Day 58 = Backend API testing with `pytest`.

Today we add automated tests for:

```text
auth
categories
expenses
filters
summary
user ownership protection
```

This is important because from now on every backend change should be tested.

## 1. Create test database

Open PostgreSQL shell:

```powershell
psql -U postgres
```

Run:

```sql
CREATE DATABASE expense_manager_test;
```

Exit:

```sql
\q
```

Do not use your main `expense_manager` database for tests.

## 2. Go to backend

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
```

Install test packages if missing:

```powershell
python -m pip install pytest httpx
```

## 3. Create `pytest.ini`

Create this file inside `backend/`:

```ini
[pytest]
pythonpath = .
testpaths = tests
```

## 4. Create `tests/conftest.py`

```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from app.db import Base, get_db
from app.main import app
from app import models

TEST_DATABASE_URL = "postgresql+psycopg://postgres:your_postgres_password@localhost:5432/expense_manager_test"

engine = create_engine(TEST_DATABASE_URL)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


@pytest.fixture()
def db_session():
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)

    session = TestingSessionLocal()

    try:
        yield session
    finally:
        session.close()


@pytest.fixture()
def client(db_session):
    def override_get_db():
        try:
            yield db_session
        finally:
            pass

    app.dependency_overrides[get_db] = override_get_db

    with TestClient(app) as test_client:
        yield test_client

    app.dependency_overrides.clear()


@pytest.fixture()
def auth_headers(client):
    client.post(
        "/auth/register",
        json={
            "username": "hari",
            "email": "hari@example.com",
            "password": "password123",
        },
    )

    response = client.post(
        "/auth/login",
        data={
            "username": "hari",
            "password": "password123",
        },
    )

    token = response.json()["access_token"]

    return {"Authorization": f"Bearer {token}"}


def create_category(client, auth_headers, name="Food"):
    response = client.post(
        "/categories",
        json={"name": name},
        headers=auth_headers,
    )

    assert response.status_code == 201
    return response.json()


def create_expense(client, auth_headers, title="Breakfast", amount="120.50", category_id=1):
    response = client.post(
        "/expenses",
        json={
            "title": title,
            "amount": amount,
            "category_id": category_id,
        },
        headers=auth_headers,
    )

    assert response.status_code == 201
    return response.json()
```

Replace this password:

```python
your_postgres_password
```

with your real PostgreSQL password.

## 5. Create `tests/test_auth.py`

```python
def test_register_user(client):
    response = client.post(
        "/auth/register",
        json={
            "username": "john",
            "email": "john@example.com",
            "password": "password123",
        },
    )

    assert response.status_code == 201

    data = response.json()
    assert data["username"] == "john"
    assert data["email"] == "john@example.com"
    assert "hashed_password" not in data


def test_login_user(client):
    client.post(
        "/auth/register",
        json={
            "username": "john",
            "email": "john@example.com",
            "password": "password123",
        },
    )

    response = client.post(
        "/auth/login",
        data={
            "username": "john",
            "password": "password123",
        },
    )

    assert response.status_code == 200

    data = response.json()
    assert "access_token" in data
    assert data["token_type"] == "bearer"


def test_me_requires_token(client):
    response = client.get("/auth/me")

    assert response.status_code == 401
```

## 6. Create `tests/test_categories.py`

```python
def test_create_category(client, auth_headers):
    response = client.post(
        "/categories",
        json={"name": "Food"},
        headers=auth_headers,
    )

    assert response.status_code == 201

    data = response.json()
    assert data["name"] == "Food"


def test_list_categories(client, auth_headers):
    client.post("/categories", json={"name": "Food"}, headers=auth_headers)
    client.post("/categories", json={"name": "Travel"}, headers=auth_headers)

    response = client.get("/categories", headers=auth_headers)

    assert response.status_code == 200

    data = response.json()
    assert len(data) == 2


def test_duplicate_category_for_same_user_fails(client, auth_headers):
    client.post("/categories", json={"name": "Food"}, headers=auth_headers)

    response = client.post(
        "/categories",
        json={"name": "Food"},
        headers=auth_headers,
    )

    assert response.status_code == 400
```

## 7. Create `tests/test_expenses.py`

```python
from decimal import Decimal

from tests.conftest import create_category, create_expense


def test_create_expense(client, auth_headers):
    category = create_category(client, auth_headers)

    response = client.post(
        "/expenses",
        json={
            "title": "Breakfast",
            "amount": "120.50",
            "category_id": category["id"],
        },
        headers=auth_headers,
    )

    assert response.status_code == 201

    data = response.json()
    assert data["title"] == "Breakfast"
    assert Decimal(data["amount"]) == Decimal("120.50")
    assert data["category_id"] == category["id"]


def test_list_expenses(client, auth_headers):
    category = create_category(client, auth_headers)

    create_expense(client, auth_headers, "Breakfast", "120.50", category["id"])
    create_expense(client, auth_headers, "Lunch", "250.00", category["id"])

    response = client.get("/expenses", headers=auth_headers)

    assert response.status_code == 200

    data = response.json()
    assert len(data) == 2


def test_filter_expenses_by_search(client, auth_headers):
    category = create_category(client, auth_headers)

    create_expense(client, auth_headers, "Breakfast", "120.50", category["id"])
    create_expense(client, auth_headers, "Cab", "300.00", category["id"])

    response = client.get("/expenses?q=break", headers=auth_headers)

    assert response.status_code == 200

    data = response.json()
    assert len(data) == 1
    assert data[0]["title"] == "Breakfast"


def test_expense_summary(client, auth_headers):
    category = create_category(client, auth_headers)

    create_expense(client, auth_headers, "Breakfast", "100.00", category["id"])
    create_expense(client, auth_headers, "Lunch", "200.00", category["id"])

    response = client.get("/expenses/summary", headers=auth_headers)

    assert response.status_code == 200

    data = response.json()
    assert data["total_expenses"] == 2
    assert Decimal(data["total_amount"]) == Decimal("300.00")
    assert Decimal(data["average_amount"]) == Decimal("150.00")


def test_update_expense(client, auth_headers):
    category = create_category(client, auth_headers)
    expense = create_expense(client, auth_headers, "Breakfast", "120.50", category["id"])

    response = client.put(
        f"/expenses/{expense['id']}",
        json={
            "title": "Dinner",
            "amount": "500.00",
            "category_id": category["id"],
        },
        headers=auth_headers,
    )

    assert response.status_code == 200

    data = response.json()
    assert data["title"] == "Dinner"
    assert Decimal(data["amount"]) == Decimal("500.00")


def test_delete_expense(client, auth_headers):
    category = create_category(client, auth_headers)
    expense = create_expense(client, auth_headers, "Breakfast", "120.50", category["id"])

    delete_response = client.delete(
        f"/expenses/{expense['id']}",
        headers=auth_headers,
    )

    assert delete_response.status_code == 200

    get_response = client.get(
        f"/expenses/{expense['id']}",
        headers=auth_headers,
    )

    assert get_response.status_code == 404
```

## 8. Create `tests/test_ownership.py`

```python
def register_and_login(client, username, email):
    client.post(
        "/auth/register",
        json={
            "username": username,
            "email": email,
            "password": "password123",
        },
    )

    response = client.post(
        "/auth/login",
        data={
            "username": username,
            "password": "password123",
        },
    )

    token = response.json()["access_token"]

    return {"Authorization": f"Bearer {token}"}


def test_user_cannot_see_another_users_expenses(client):
    user_one_headers = register_and_login(client, "userone", "userone@example.com")
    user_two_headers = register_and_login(client, "usertwo", "usertwo@example.com")

    category_response = client.post(
        "/categories",
        json={"name": "Food"},
        headers=user_one_headers,
    )
    category_id = category_response.json()["id"]

    expense_response = client.post(
        "/expenses",
        json={
            "title": "Private Expense",
            "amount": "999.00",
            "category_id": category_id,
        },
        headers=user_one_headers,
    )
    expense_id = expense_response.json()["id"]

    list_response = client.get("/expenses", headers=user_two_headers)

    assert list_response.status_code == 200
    assert list_response.json() == []

    get_response = client.get(f"/expenses/{expense_id}", headers=user_two_headers)

    assert get_response.status_code == 404


def test_user_cannot_create_expense_with_another_users_category(client):
    user_one_headers = register_and_login(client, "userone", "userone@example.com")
    user_two_headers = register_and_login(client, "usertwo", "usertwo@example.com")

    category_response = client.post(
        "/categories",
        json={"name": "Food"},
        headers=user_one_headers,
    )
    category_id = category_response.json()["id"]

    response = client.post(
        "/expenses",
        json={
            "title": "Wrong Category Expense",
            "amount": "500.00",
            "category_id": category_id,
        },
        headers=user_two_headers,
    )

    assert response.status_code == 404
```

## 9. Run tests

```powershell
pytest
```

Expected result:

```text
12 passed
```

If you want detailed output:

```powershell
pytest -v
```

## 10. Fix common errors

If you get this:

```text
database "expense_manager_test" does not exist
```

Create test DB again:

```powershell
psql -U postgres -c "CREATE DATABASE expense_manager_test;"
```

If you get this:

```text
password authentication failed
```

Fix `TEST_DATABASE_URL` password in `tests/conftest.py`.

If `/expenses/summary` gives `422`, check route order in `app/routers/expenses.py`.

This must come before `/{expense_id}`:

```python
@router.get("/summary")
```

## 11. Commit

```powershell
git status
git add .
git commit -m "Day 58 add backend API tests"
```

Senior concept for today: tests should verify business rules, not just happy paths. The important test today is ownership protection: one user must never access another user’s expenses.

Sources checked: [FastAPI testing dependency overrides](https://fastapi.tiangolo.com/advanced/testing-dependencies/), [SQLAlchemy session rollback behavior](https://docs.sqlalchemy.org/en/20/orm/session_basics.html), [pytest fixtures](https://docs.pytest.org/en/6.2.x/fixture.html).
