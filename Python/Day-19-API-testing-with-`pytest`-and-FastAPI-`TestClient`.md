Day 19 is API testing with `pytest` and FastAPI `TestClient`. This is a serious backend skill: your API should not be trusted just because Swagger works manually.

**Day 19 Goal**
You should understand:

- Automated API tests
- FastAPI `TestClient`
- Testing `GET`, `POST`, `PUT`, `DELETE`
- Dependency override for test database
- Why tests should not touch your real PostgreSQL database

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-19-fastapi-testing
cd day-19-fastapi-testing
code .
```

Use your Day 18 project structure:

```text
day-19-fastapi-testing/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── db.py
│   ├── models.py
│   ├── schemas.py
│   └── routers/
│       ├── __init__.py
│       ├── categories.py
│       └── expenses.py
├── tests/
│   ├── __init__.py
│   └── test_api.py
├── .env
├── .gitignore
└── requirements.txt
```

Install test packages:

```powershell
python -m pip install pytest httpx
```

Add these to `requirements.txt`:

```text
pytest
httpx
```

Create `tests/test_api.py`:

```python
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.pool import StaticPool
from sqlalchemy.orm import sessionmaker

from app.db import Base, get_db
from app.main import app


TEST_DATABASE_URL = "sqlite://"

engine = create_engine(
    TEST_DATABASE_URL,
    connect_args={"check_same_thread": False},
    poolclass=StaticPool,
)

TestingSessionLocal = sessionmaker(bind=engine)

Base.metadata.create_all(bind=engine)


def override_get_db():
    db = TestingSessionLocal()

    try:
        yield db
    finally:
        db.close()


app.dependency_overrides[get_db] = override_get_db

client = TestClient(app)


def test_health_check():
    response = client.get("/health")

    assert response.status_code == 200
    assert response.json() == {"status": "ok"}


def test_create_category():
    response = client.post(
        "/categories",
        json={"name": "Food"},
    )

    assert response.status_code == 201
    assert response.json()["name"] == "Food"


def test_list_categories():
    client.post("/categories", json={"name": "Travel"})

    response = client.get("/categories")

    assert response.status_code == 200
    assert len(response.json()) >= 1


def test_create_expense():
    category_response = client.post("/categories", json={"name": "Study"})
    category_id = category_response.json()["id"]

    response = client.post(
        "/expenses",
        json={
            "title": "Python book",
            "amount": 500,
            "category_id": category_id,
        },
    )

    assert response.status_code == 201
    assert response.json()["title"] == "Python book"
    assert response.json()["category"]["name"] == "Study"


def test_create_expense_with_invalid_category():
    response = client.post(
        "/expenses",
        json={
            "title": "Tea",
            "amount": 20,
            "category_id": 999,
        },
    )

    assert response.status_code == 404
    assert response.json()["detail"] == "Category not found"


def test_delete_expense():
    category_response = client.post("/categories", json={"name": "Bills"})
    category_id = category_response.json()["id"]

    expense_response = client.post(
        "/expenses",
        json={
            "title": "Internet bill",
            "amount": 999,
            "category_id": category_id,
        },
    )
    expense_id = expense_response.json()["id"]

    delete_response = client.delete(f"/expenses/{expense_id}")

    assert delete_response.status_code == 200
    assert delete_response.json() == {"message": "Expense deleted"}


def test_delete_missing_expense():
    response = client.delete("/expenses/999")

    assert response.status_code == 404
    assert response.json()["detail"] == "Expense not found"
```

Run tests:

```powershell
pytest
```

Expected result:

```text
7 passed
```

Run formatter and linter:

```powershell
black .
ruff check .
```

**Important Concept**
In your real app, `get_db()` connects to PostgreSQL.

In tests, this line replaces it:

```python
app.dependency_overrides[get_db] = override_get_db
```

That means tests use a temporary SQLite database instead of your real PostgreSQL database. This keeps tests fast and prevents test data from polluting your real database.

**Challenge**
Add one test for your Day 18 challenge endpoint:

```text
PUT /expenses/{expense_id}
```

Test these cases:

- Update expense with valid category
- Update expense with missing expense ID returns `404`
- Update expense with invalid category ID returns `404`

Commit:

```powershell
git add .
git commit -m "Day 19 add automated API tests"
```

Day 19 is complete when you can explain why manual Swagger testing is useful but automated tests are required for professional backend work.
