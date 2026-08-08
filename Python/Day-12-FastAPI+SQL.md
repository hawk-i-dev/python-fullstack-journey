Day 12 is FastAPI + PostgreSQL. Today your API stops storing data in a Python list and starts using a real database.

**Day 12 Goal**
You should understand:

- FastAPI route calls database function
- PostgreSQL stores API data permanently
- `POST`, `GET`, `PUT`, `DELETE` with database
- Why API layer and database layer should be separate
- Why we use parameterized SQL

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-12-fastapi-postgres
cd day-12-fastapi-postgres
code .
```

Install packages:

```powershell
python -m pip install fastapi uvicorn psycopg[binary]
```

Create database in `psql`:

```sql
CREATE DATABASE python_fullstack_day12;
\c python_fullstack_day12
```

Create table:

```sql
CREATE TABLE expenses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    category VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Create `db.py`:

```python
import psycopg

DB_NAME = "python_fullstack_day12"
DB_USER = "postgres"
DB_PASSWORD = "your_password_here"
DB_HOST = "localhost"
DB_PORT = 5432


def get_connection():
    return psycopg.connect(
        dbname=DB_NAME,
        user=DB_USER,
        password=DB_PASSWORD,
        host=DB_HOST,
        port=DB_PORT,
    )
```

Create `schemas.py`:

```python
from pydantic import BaseModel, Field


class ExpenseCreate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category: str = Field(min_length=1)


class ExpenseUpdate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category: str = Field(min_length=1)
```

Create `expense_repository.py`:

```python
from db import get_connection


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
            row = cursor.fetchone()
            return row_to_expense(row)


def get_all_expenses():
    sql = """
        SELECT id, title, amount, category, created_at
        FROM expenses
        ORDER BY id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)
            rows = cursor.fetchall()
            return [row_to_expense(row) for row in rows]


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

            if row is None:
                return None

            return row_to_expense(row)


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

            if row is None:
                return None

            return row_to_expense(row)


def delete_expense(expense_id):
    sql = """
        DELETE FROM expenses
        WHERE id = %s
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (expense_id,))
            return cursor.rowcount
```

Create `main.py`:

```python
from fastapi import FastAPI, HTTPException

from expense_repository import (
    create_expense,
    delete_expense,
    get_all_expenses,
    get_expense_by_id,
    update_expense,
)
from schemas import ExpenseCreate, ExpenseUpdate

app = FastAPI(title="Expense API With PostgreSQL")


@app.get("/")
def home():
    return {"message": "Expense API with PostgreSQL is running"}


@app.get("/expenses")
def list_expenses():
    return get_all_expenses()


@app.get("/expenses/{expense_id}")
def get_expense(expense_id: int):
    expense = get_expense_by_id(expense_id)

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    return expense


@app.post("/expenses", status_code=201)
def add_expense(expense_data: ExpenseCreate):
    return create_expense(
        expense_data.title,
        expense_data.amount,
        expense_data.category,
    )


@app.put("/expenses/{expense_id}")
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

Run server:

```powershell
uvicorn main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test in this order:

1. `GET /`
2. `GET /expenses`
3. `POST /expenses`
4. `GET /expenses`
5. `GET /expenses/1`
6. `PUT /expenses/1`
7. `DELETE /expenses/1`
8. `GET /expenses/1`

Use this JSON for `POST`:

```json
{
  "title": "Tea",
  "amount": 20,
  "category": "Food"
}
```

**Challenge**
Add this endpoint:

```python
@app.get("/expenses/category/{category_name}")
def get_expenses_by_category(category_name: str):
    pass
```

You need to create a repository function that runs:

```sql
SELECT id, title, amount, category, created_at
FROM expenses
WHERE LOWER(category) = LOWER(%s)
ORDER BY id
```

Commit:

```powershell
git status
git add .
git commit -m "Day 12 connect FastAPI API to PostgreSQL"
```

Day 12 is complete when you can explain this flow:

`Browser / Swagger UI -> FastAPI route -> repository function -> PostgreSQL -> repository returns data -> FastAPI returns JSON`.
