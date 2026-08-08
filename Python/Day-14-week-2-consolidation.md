Day 14 is Week 2 consolidation. Today you turn your API into a cleaner mini backend project with database setup, health check, totals, and category reports.

**Day 14 Goal**
You should be able to explain and build:

- FastAPI project structure
- PostgreSQL CRUD
- Environment-based config
- API response models
- Basic reporting endpoints
- Database setup script
- Manual API testing using Swagger UI

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-14-expense-api-review
cd day-14-expense-api-review
code .
```

Install packages:

```powershell
python -m pip install fastapi uvicorn psycopg[binary] pydantic-settings
```

Create this structure:

```text
day-14-expense-api-review/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── db.py
│   ├── schemas.py
│   └── expense_repository.py
├── scripts/
│   └── setup_db.sql
├── .env
├── .env.example
├── .gitignore
└── requirements.txt
```

Create `.env`:

```env
DB_NAME=python_fullstack_day14
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
```

Create `.env.example`:

```env
DB_NAME=python_fullstack_day14
DB_USER=postgres
DB_PASSWORD=change_me
DB_HOST=localhost
DB_PORT=5432
```

Create `.gitignore`:

```gitignore
.venv/
__pycache__/
.env
*.pyc
.pytest_cache/
```

Create `requirements.txt`:

```text
fastapi
uvicorn
psycopg[binary]
pydantic-settings
```

Create `scripts/setup_db.sql`:

```sql
CREATE TABLE IF NOT EXISTS expenses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    category VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

In `psql`, run:

```sql
CREATE DATABASE python_fullstack_day14;
\c python_fullstack_day14
\i 'C:/Users/YourName/Documents/python-fullstack-journey/day-14-expense-api-review/scripts/setup_db.sql'
```

Replace `YourName` with your Windows username.

Create `app/config.py`:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    db_name: str
    db_user: str
    db_password: str
    db_host: str
    db_port: int

    model_config = SettingsConfigDict(env_file=".env")


settings = Settings()
```

Create `app/db.py`:

```python
import psycopg

from app.config import settings


def get_connection():
    return psycopg.connect(
        dbname=settings.db_name,
        user=settings.db_user,
        password=settings.db_password,
        host=settings.db_host,
        port=settings.db_port,
    )
```

Create `app/schemas.py`:

```python
from datetime import datetime

from pydantic import BaseModel, Field


class ExpenseCreate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category: str = Field(min_length=1)


class ExpenseUpdate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category: str = Field(min_length=1)


class ExpenseResponse(BaseModel):
    id: int
    title: str
    amount: float
    category: str
    created_at: datetime


class TotalResponse(BaseModel):
    total: float


class CategoryTotalResponse(BaseModel):
    category: str
    total: float
```

Create `app/expense_repository.py`:

```python
from app.db import get_connection


def row_to_expense(row):
    return {
        "id": row[0],
        "title": row[1],
        "amount": float(row[2]),
        "category": row[3],
        "created_at": row[4],
    }


def create_expense(title, amount, category):
    sql = """
        INSERT INTO expenses (title, amount, category)
        VALUES (%s, %s, %s)
        RETURNING id, title, amount, category, created_at
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (title, amount, category))
            return row_to_expense(cursor.fetchone())


def get_all_expenses():
    sql = """
        SELECT id, title, amount, category, created_at
        FROM expenses
        ORDER BY id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)
            return [row_to_expense(row) for row in cursor.fetchall()]


def get_expense_by_id(expense_id):
    sql = """
        SELECT id, title, amount, category, created_at
        FROM expenses
        WHERE id = %s
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (expense_id,))
            row = cursor.fetchone()
            return None if row is None else row_to_expense(row)


def get_expenses_by_category(category):
    sql = """
        SELECT id, title, amount, category, created_at
        FROM expenses
        WHERE LOWER(category) = LOWER(%s)
        ORDER BY id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (category,))
            return [row_to_expense(row) for row in cursor.fetchall()]


def update_expense(expense_id, title, amount, category):
    sql = """
        UPDATE expenses
        SET title = %s, amount = %s, category = %s
        WHERE id = %s
        RETURNING id, title, amount, category, created_at
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (title, amount, category, expense_id))
            row = cursor.fetchone()
            return None if row is None else row_to_expense(row)


def delete_expense(expense_id):
    sql = """
        DELETE FROM expenses
        WHERE id = %s
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (expense_id,))
            return cursor.rowcount


def get_total_expense():
    sql = """
        SELECT COALESCE(SUM(amount), 0)
        FROM expenses
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)
            return float(cursor.fetchone()[0])


def get_total_by_category():
    sql = """
        SELECT category, SUM(amount)
        FROM expenses
        GROUP BY category
        ORDER BY SUM(amount) DESC
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)

            results = []
            for row in cursor.fetchall():
                results.append(
                    {
                        "category": row[0],
                        "total": float(row[1]),
                    }
                )

            return results
```

Create `app/main.py`:

```python
from fastapi import FastAPI, HTTPException

from app.expense_repository import (
    create_expense,
    delete_expense,
    get_all_expenses,
    get_expense_by_id,
    get_expenses_by_category,
    get_total_by_category,
    get_total_expense,
    update_expense,
)
from app.schemas import (
    CategoryTotalResponse,
    ExpenseCreate,
    ExpenseResponse,
    ExpenseUpdate,
    TotalResponse,
)

app = FastAPI(title="Expense API Review")


@app.get("/")
def home():
    return {"message": "Expense API Review is running"}


@app.get("/health")
def health_check():
    return {"status": "ok"}


@app.get("/expenses/total", response_model=TotalResponse)
def expense_total():
    return {"total": get_total_expense()}


@app.get("/expenses/reports/by-category", response_model=list[CategoryTotalResponse])
def expense_total_by_category():
    return get_total_by_category()


@app.get("/expenses", response_model=list[ExpenseResponse])
def list_expenses():
    return get_all_expenses()


@app.get("/expenses/category/{category_name}", response_model=list[ExpenseResponse])
def list_expenses_by_category(category_name: str):
    return get_expenses_by_category(category_name)


@app.get("/expenses/{expense_id}", response_model=ExpenseResponse)
def get_expense(expense_id: int):
    expense = get_expense_by_id(expense_id)

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    return expense


@app.post("/expenses", response_model=ExpenseResponse, status_code=201)
def add_expense(expense_data: ExpenseCreate):
    return create_expense(
        expense_data.title,
        expense_data.amount,
        expense_data.category,
    )


@app.put("/expenses/{expense_id}", response_model=ExpenseResponse)
def edit_expense(expense_id: int, expense_data: ExpenseUpdate):
    expense = update_expense(
        expense_id,
        expense_data.title,
        expense_data.amount,
        expense_data.category,
    )

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    return expense


@app.delete("/expenses/{expense_id}")
def remove_expense(expense_id: int):
    deleted_count = delete_expense(expense_id)

    if deleted_count == 0:
        raise HTTPException(status_code=404, detail="Expense not found")

    return {"message": "Expense deleted"}
```

Run:

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test these endpoints:

- `GET /health`
- `POST /expenses`
- `GET /expenses`
- `GET /expenses/total`
- `GET /expenses/reports/by-category`
- `GET /expenses/category/Food`
- `PUT /expenses/{expense_id}`
- `DELETE /expenses/{expense_id}`

**Important Routing Detail**
Keep these routes above `/expenses/{expense_id}`:

```python
@app.get("/expenses/total")
@app.get("/expenses/reports/by-category")
@app.get("/expenses/category/{category_name}")
```

Reason: path routes like `/expenses/{expense_id}` can accidentally catch fixed paths like `/expenses/total` if ordered badly.

Commit:

```powershell
git status
git add .
git commit -m "Day 14 consolidate FastAPI PostgreSQL API"
```

Day 14 is complete when you can explain the full request flow from Swagger UI to FastAPI to PostgreSQL and back as JSON.
