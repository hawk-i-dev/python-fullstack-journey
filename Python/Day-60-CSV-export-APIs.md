Day 60 = CSV export APIs.

We will add:

```text
GET /exports/expenses.csv
GET /exports/category-summary.csv
GET /exports/monthly-summary.csv
```

No database migration required.

## 1. Go to backend

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
```

## 2. Create `app/routers/exports.py`

```python
import csv
from datetime import date, datetime, timedelta
from decimal import Decimal
from io import StringIO
from typing import Literal

from fastapi import APIRouter, Depends, HTTPException, Query
from fastapi.responses import Response
from sqlalchemy import func, select
from sqlalchemy.orm import Session, selectinload

from app.db import get_db
from app.models import Category, Expense, User
from app.security import get_current_user

router = APIRouter(prefix="/exports", tags=["exports"])


def safe_csv_value(value):
    if value is None:
        return ""

    text = str(value)

    if text.startswith(("=", "+", "-", "@")):
        return "'" + text

    return text


def build_csv_response(filename: str, fieldnames: list[str], rows: list[dict]):
    output = StringIO()
    writer = csv.DictWriter(output, fieldnames=fieldnames)

    writer.writeheader()

    for row in rows:
        writer.writerow({field: safe_csv_value(row.get(field)) for field in fieldnames})

    return Response(
        content=output.getvalue(),
        media_type="text/csv",
        headers={
            "Content-Disposition": f'attachment; filename="{filename}"',
        },
    )


def ensure_owned_category(db: Session, category_id: int, user_id: int):
    category = db.scalar(
        select(Category).where(
            Category.id == category_id,
            Category.user_id == user_id,
        )
    )

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")


def apply_export_filters(
    statement,
    db: Session,
    user_id: int,
    q: str | None,
    category_id: int | None,
    start_date: date | None,
    end_date: date | None,
    min_amount: Decimal | None,
    max_amount: Decimal | None,
):
    statement = statement.where(Expense.user_id == user_id)

    if q:
        statement = statement.where(Expense.title.ilike(f"%{q.strip()}%"))

    if category_id is not None:
        ensure_owned_category(db, category_id, user_id)
        statement = statement.where(Expense.category_id == category_id)

    if start_date and end_date and end_date < start_date:
        raise HTTPException(status_code=400, detail="end_date cannot be before start_date")

    if start_date:
        start_datetime = datetime.combine(start_date, datetime.min.time())
        statement = statement.where(Expense.created_at >= start_datetime)

    if end_date:
        next_day = end_date + timedelta(days=1)
        end_datetime = datetime.combine(next_day, datetime.min.time())
        statement = statement.where(Expense.created_at < end_datetime)

    if min_amount is not None and max_amount is not None and max_amount < min_amount:
        raise HTTPException(status_code=400, detail="max_amount cannot be less than min_amount")

    if min_amount is not None:
        statement = statement.where(Expense.amount >= min_amount)

    if max_amount is not None:
        statement = statement.where(Expense.amount <= max_amount)

    return statement


@router.get("/expenses.csv")
def export_expenses_csv(
    q: str | None = Query(default=None, max_length=100),
    category_id: int | None = Query(default=None, gt=0),
    start_date: date | None = None,
    end_date: date | None = None,
    min_amount: Decimal | None = Query(default=None, gt=0),
    max_amount: Decimal | None = Query(default=None, gt=0),
    sort_by: Literal["created_at", "amount", "title"] = "created_at",
    sort_order: Literal["asc", "desc"] = "desc",
    max_rows: int = Query(default=5000, ge=1, le=50000),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    statement = select(Expense).options(selectinload(Expense.category))

    statement = apply_export_filters(
        statement,
        db,
        current_user.id,
        q,
        category_id,
        start_date,
        end_date,
        min_amount,
        max_amount,
    )

    sort_columns = {
        "created_at": Expense.created_at,
        "amount": Expense.amount,
        "title": Expense.title,
    }

    sort_column = sort_columns[sort_by]

    if sort_order == "desc":
        statement = statement.order_by(sort_column.desc(), Expense.id.desc())
    else:
        statement = statement.order_by(sort_column.asc(), Expense.id.asc())

    expenses = db.scalars(statement.limit(max_rows)).all()

    rows = [
        {
            "id": expense.id,
            "title": expense.title,
            "amount": expense.amount,
            "category_id": expense.category_id,
            "category_name": expense.category.name if expense.category else "",
            "created_at": expense.created_at.isoformat() if expense.created_at else "",
        }
        for expense in expenses
    ]

    return build_csv_response(
        "expenses.csv",
        ["id", "title", "amount", "category_id", "category_name", "created_at"],
        rows,
    )


@router.get("/category-summary.csv")
def export_category_summary_csv(
    start_date: date | None = None,
    end_date: date | None = None,
    category_id: int | None = Query(default=None, gt=0),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    statement = (
        select(
            Category.id.label("category_id"),
            Category.name.label("category_name"),
            func.count(Expense.id).label("expense_count"),
            func.coalesce(func.sum(Expense.amount), Decimal("0.00")).label("total_amount"),
            func.coalesce(func.avg(Expense.amount), Decimal("0.00")).label("average_amount"),
        )
        .join(Category, Expense.category_id == Category.id)
        .group_by(Category.id, Category.name)
        .order_by(func.sum(Expense.amount).desc())
    )

    statement = apply_export_filters(
        statement,
        db,
        current_user.id,
        None,
        category_id,
        start_date,
        end_date,
        None,
        None,
    )

    result = db.execute(statement).all()

    rows = [
        {
            "category_id": row.category_id,
            "category_name": row.category_name,
            "expense_count": row.expense_count,
            "total_amount": row.total_amount,
            "average_amount": row.average_amount,
        }
        for row in result
    ]

    return build_csv_response(
        "category-summary.csv",
        ["category_id", "category_name", "expense_count", "total_amount", "average_amount"],
        rows,
    )


@router.get("/monthly-summary.csv")
def export_monthly_summary_csv(
    start_date: date | None = None,
    end_date: date | None = None,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    month_expression = func.date_trunc("month", Expense.created_at)

    statement = (
        select(
            month_expression.label("month"),
            func.count(Expense.id).label("expense_count"),
            func.coalesce(func.sum(Expense.amount), Decimal("0.00")).label("total_amount"),
        )
        .group_by(month_expression)
        .order_by(month_expression.asc())
    )

    statement = apply_export_filters(
        statement,
        db,
        current_user.id,
        None,
        None,
        start_date,
        end_date,
        None,
        None,
    )

    result = db.execute(statement).all()

    rows = [
        {
            "month": row.month.strftime("%Y-%m"),
            "expense_count": row.expense_count,
            "total_amount": row.total_amount,
        }
        for row in result
    ]

    return build_csv_response(
        "monthly-summary.csv",
        ["month", "expense_count", "total_amount"],
        rows,
    )
```

## 3. Update `app/main.py`

Update imports:

```python
from app.routers import auth, categories, expenses, reports, exports
```

Add:

```python
app.include_router(exports.router)
```

Router section:

```python
app.include_router(auth.router)
app.include_router(categories.router)
app.include_router(expenses.router)
app.include_router(reports.router)
app.include_router(exports.router)
```

## 4. Run server

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

## 5. Test in Swagger

After login and `Authorize`, test:

```text
GET /exports/expenses.csv
GET /exports/expenses.csv?q=breakfast
GET /exports/expenses.csv?category_id=1
GET /exports/expenses.csv?start_date=2026-09-01&end_date=2026-09-30
GET /exports/category-summary.csv
GET /exports/monthly-summary.csv
```

Browser should download a `.csv` file.

## 6. Add tests: `tests/test_exports.py`

```python
import csv
from io import StringIO

from tests.conftest import create_category, create_expense


def parse_csv_response(response):
    return list(csv.DictReader(StringIO(response.text)))


def test_export_expenses_csv(client, auth_headers):
    category = create_category(client, auth_headers, "Food")
    create_expense(client, auth_headers, "Breakfast", "120.50", category["id"])

    response = client.get("/exports/expenses.csv", headers=auth_headers)

    assert response.status_code == 200
    assert response.headers["content-type"].startswith("text/csv")
    assert "attachment" in response.headers["content-disposition"]

    rows = parse_csv_response(response)

    assert len(rows) == 1
    assert rows[0]["title"] == "Breakfast"
    assert rows[0]["amount"] == "120.50"
    assert rows[0]["category_name"] == "Food"


def test_export_category_summary_csv(client, auth_headers):
    category = create_category(client, auth_headers, "Food")
    create_expense(client, auth_headers, "Breakfast", "100.00", category["id"])
    create_expense(client, auth_headers, "Lunch", "200.00", category["id"])

    response = client.get("/exports/category-summary.csv", headers=auth_headers)

    assert response.status_code == 200

    rows = parse_csv_response(response)

    assert len(rows) == 1
    assert rows[0]["category_name"] == "Food"
    assert rows[0]["expense_count"] == "2"
    assert rows[0]["total_amount"] == "300.00"


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


def test_export_does_not_include_another_users_expenses(client):
    user_one_headers = register_and_login(client, "userone", "userone@example.com")
    user_two_headers = register_and_login(client, "usertwo", "usertwo@example.com")

    category_response = client.post(
        "/categories",
        json={"name": "Food"},
        headers=user_one_headers,
    )

    category_id = category_response.json()["id"]

    client.post(
        "/expenses",
        json={
            "title": "Private Expense",
            "amount": "999.00",
            "category_id": category_id,
        },
        headers=user_one_headers,
    )

    response = client.get("/exports/expenses.csv", headers=user_two_headers)

    assert response.status_code == 200

    rows = parse_csv_response(response)

    assert rows == []
```

## 7. Run tests

```powershell
pytest
```

Expected:

```text
18 passed
```

Your number may be higher if you added extra tests.

## 8. Commit

```powershell
git status
git add .
git commit -m "Day 60 add CSV export APIs"
```

Senior concept today: export APIs are high-risk because they can leak bulk data. Every export query must include user ownership filtering:

```python
Expense.user_id == current_user.id
```

Also note the `safe_csv_value()` function. It prevents CSV formula injection when someone opens the exported file in Excel.

Sources checked: [FastAPI custom responses](https://fastapi.tiangolo.com/advanced/custom-response/), [FastAPI response headers](https://fastapi.tiangolo.com/advanced/response-headers/), [Python CSV DictWriter](https://docs.python.org/3/library/csv.html).
