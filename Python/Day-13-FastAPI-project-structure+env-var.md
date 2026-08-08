Day 13 is professional FastAPI project structure + environment variables. Today we clean up the Day 12 API so it looks closer to real backend code.

**Day 13 Goal**
You should understand:

- Why backend projects are split into folders
- Why database passwords should not be hardcoded
- What `.env` is
- What `.gitignore` is
- How FastAPI apps are structured professionally
- How `requirements.txt` helps recreate a project

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-13-fastapi-structure
cd day-13-fastapi-structure
code .
```

Install packages:

```powershell
python -m pip install fastapi uvicorn psycopg[binary] pydantic-settings
```

Create this structure:

```text
day-13-fastapi-structure/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── db.py
│   ├── schemas.py
│   └── expense_repository.py
├── .env
├── .gitignore
└── requirements.txt
```

Create `.env`:

```env
DB_NAME=python_fullstack_day12
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
```

Create `.gitignore`:

```gitignore
.venv/
__pycache__/
.env
*.pyc
```

Important: `.env` contains your password, so never commit it.

Create `requirements.txt`:

```text
fastapi
uvicorn
psycopg[binary]
pydantic-settings
```

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
    update_expense,
)
from app.schemas import ExpenseCreate, ExpenseResponse, ExpenseUpdate

app = FastAPI(title="Professional Expense API")


@app.get("/")
def home():
    return {"message": "Professional Expense API is running"}


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

Run the API from the project root:

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test:

- `GET /`
- `POST /expenses`
- `GET /expenses`
- `GET /expenses/category/Food`
- `GET /expenses/1`
- `PUT /expenses/1`
- `DELETE /expenses/1`

**Important Concept**

Before Day 13:

```text
main.py had too much responsibility
db.py had hardcoded password
response structure was not clearly documented
```

After Day 13:

```text
config.py reads environment variables
db.py only handles database connection
expense_repository.py handles SQL
schemas.py defines request/response models
main.py handles API routes
```

**Challenge**
Add one new endpoint:

```text
GET /expenses/total
```

It should return:

```json
{
  "total": 250.5
}
```

Hint: create a repository function using:

```sql
SELECT COALESCE(SUM(amount), 0)
FROM expenses
```

Commit:

```powershell
git status
git add .
git commit -m "Day 13 structure FastAPI project professionally"
```

Day 13 is complete when you can explain why `.env` should not be committed and why `response_model` is useful.
