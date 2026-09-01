Day 59 = Reports API.

Today we add analytics endpoints for the backend:

```text
GET /reports/category-summary
GET /reports/monthly-summary
GET /reports/dashboard
```

No database migration needed.

## 1. Go to backend

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
```

## 2. Update `app/schemas.py`

Add these schemas:

```python
class CategoryReportResponse(BaseModel):
    category_id: int
    category_name: str
    expense_count: int
    total_amount: Decimal
    average_amount: Decimal


class MonthlyReportResponse(BaseModel):
    month: str
    expense_count: int
    total_amount: Decimal


class DashboardReportResponse(BaseModel):
    total_expenses: int
    total_amount: Decimal
    average_amount: Decimal
    current_month_total: Decimal
    top_category: str | None
    top_category_amount: Decimal
```

Make sure this import already exists:

```python
from decimal import Decimal
```

## 3. Create `app/routers/reports.py`

```python
from datetime import date, datetime, timedelta
from decimal import Decimal

from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy import func, select
from sqlalchemy.orm import Session

from app.db import get_db
from app.models import Category, Expense, User
from app.schemas import (
    CategoryReportResponse,
    DashboardReportResponse,
    MonthlyReportResponse,
)
from app.security import get_current_user

router = APIRouter(prefix="/reports", tags=["reports"])


def apply_date_filters(statement, start_date: date | None, end_date: date | None):
    if start_date and end_date and end_date < start_date:
        raise HTTPException(status_code=400, detail="end_date cannot be before start_date")

    if start_date:
        start_datetime = datetime.combine(start_date, datetime.min.time())
        statement = statement.where(Expense.created_at >= start_datetime)

    if end_date:
        next_day = end_date + timedelta(days=1)
        end_datetime = datetime.combine(next_day, datetime.min.time())
        statement = statement.where(Expense.created_at < end_datetime)

    return statement


def ensure_owned_category(db: Session, category_id: int, user_id: int):
    category = db.scalar(
        select(Category).where(
            Category.id == category_id,
            Category.user_id == user_id,
        )
    )

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")


@router.get("/category-summary", response_model=list[CategoryReportResponse])
def get_category_summary(
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
        .where(Expense.user_id == current_user.id)
        .group_by(Category.id, Category.name)
        .order_by(func.sum(Expense.amount).desc())
    )

    if category_id is not None:
        ensure_owned_category(db, category_id, current_user.id)
        statement = statement.where(Expense.category_id == category_id)

    statement = apply_date_filters(statement, start_date, end_date)

    rows = db.execute(statement).all()

    return [
        {
            "category_id": row.category_id,
            "category_name": row.category_name,
            "expense_count": row.expense_count,
            "total_amount": row.total_amount,
            "average_amount": row.average_amount,
        }
        for row in rows
    ]


@router.get("/monthly-summary", response_model=list[MonthlyReportResponse])
def get_monthly_summary(
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
        .where(Expense.user_id == current_user.id)
        .group_by(month_expression)
        .order_by(month_expression.asc())
    )

    statement = apply_date_filters(statement, start_date, end_date)

    rows = db.execute(statement).all()

    return [
        {
            "month": row.month.strftime("%Y-%m"),
            "expense_count": row.expense_count,
            "total_amount": row.total_amount,
        }
        for row in rows
    ]


@router.get("/dashboard", response_model=DashboardReportResponse)
def get_dashboard_report(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    today = date.today()
    current_month_start = today.replace(day=1)
    next_month_start = datetime.combine(current_month_start, datetime.min.time()) + timedelta(days=32)
    next_month_start = next_month_start.replace(day=1)

    total_statement = select(
        func.count(Expense.id),
        func.coalesce(func.sum(Expense.amount), Decimal("0.00")),
        func.coalesce(func.avg(Expense.amount), Decimal("0.00")),
    ).where(Expense.user_id == current_user.id)

    total_expenses, total_amount, average_amount = db.execute(total_statement).one()

    current_month_statement = select(
        func.coalesce(func.sum(Expense.amount), Decimal("0.00"))
    ).where(
        Expense.user_id == current_user.id,
        Expense.created_at >= datetime.combine(current_month_start, datetime.min.time()),
        Expense.created_at < next_month_start,
    )

    current_month_total = db.scalar(current_month_statement)

    top_category_statement = (
        select(
            Category.name,
            func.coalesce(func.sum(Expense.amount), Decimal("0.00")).label("category_total"),
        )
        .join(Category, Expense.category_id == Category.id)
        .where(Expense.user_id == current_user.id)
        .group_by(Category.name)
        .order_by(func.sum(Expense.amount).desc())
        .limit(1)
    )

    top_category_row = db.execute(top_category_statement).first()

    return {
        "total_expenses": total_expenses,
        "total_amount": total_amount,
        "average_amount": average_amount,
        "current_month_total": current_month_total,
        "top_category": top_category_row.name if top_category_row else None,
        "top_category_amount": top_category_row.category_total if top_category_row else Decimal("0.00"),
    }
```

## 4. Update `app/main.py`

Update router imports:

```python
from app.routers import auth, categories, expenses, reports
```

Add:

```python
app.include_router(reports.router)
```

Final router section should look like:

```python
app.include_router(auth.router)
app.include_router(categories.router)
app.include_router(expenses.router)
app.include_router(reports.router)
```

## 5. Run server

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

## 6. Test in Swagger

Login, click `Authorize`, then test:

```text
GET /reports/category-summary
GET /reports/category-summary?start_date=2026-09-01&end_date=2026-09-30
GET /reports/category-summary?category_id=1
GET /reports/monthly-summary
GET /reports/dashboard
```

## 7. Add tests: `tests/test_reports.py`

```python
from decimal import Decimal
from datetime import date

from tests.conftest import create_category, create_expense


def test_category_summary(client, auth_headers):
    food = create_category(client, auth_headers, "Food")
    travel = create_category(client, auth_headers, "Travel")

    create_expense(client, auth_headers, "Breakfast", "100.00", food["id"])
    create_expense(client, auth_headers, "Lunch", "200.00", food["id"])
    create_expense(client, auth_headers, "Cab", "300.00", travel["id"])

    response = client.get("/reports/category-summary", headers=auth_headers)

    assert response.status_code == 200

    data = response.json()
    assert len(data) == 2

    food_report = next(item for item in data if item["category_name"] == "Food")

    assert food_report["expense_count"] == 2
    assert Decimal(food_report["total_amount"]) == Decimal("300.00")


def test_monthly_summary(client, auth_headers):
    food = create_category(client, auth_headers, "Food")
    create_expense(client, auth_headers, "Breakfast", "100.00", food["id"])

    response = client.get("/reports/monthly-summary", headers=auth_headers)

    assert response.status_code == 200

    data = response.json()
    assert len(data) == 1
    assert data[0]["month"] == date.today().strftime("%Y-%m")
    assert data[0]["expense_count"] == 1
    assert Decimal(data[0]["total_amount"]) == Decimal("100.00")


def test_dashboard_report(client, auth_headers):
    food = create_category(client, auth_headers, "Food")
    travel = create_category(client, auth_headers, "Travel")

    create_expense(client, auth_headers, "Breakfast", "100.00", food["id"])
    create_expense(client, auth_headers, "Cab", "500.00", travel["id"])

    response = client.get("/reports/dashboard", headers=auth_headers)

    assert response.status_code == 200

    data = response.json()
    assert data["total_expenses"] == 2
    assert Decimal(data["total_amount"]) == Decimal("600.00")
    assert data["top_category"] == "Travel"
    assert Decimal(data["top_category_amount"]) == Decimal("500.00")
```

## 8. Run tests

```powershell
pytest
```

Expected:

```text
15 passed
```

Your exact number may be higher if you added extra tests earlier.

## 9. Commit

```powershell
git status
git add .
git commit -m "Day 59 add expense reports API"
```

Senior concept today: reports should be calculated in the database using `COUNT`, `SUM`, `AVG`, `GROUP BY`, and date functions. Pulling all rows into Python and looping is inefficient once data grows.

Sources checked: [FastAPI query parameters](https://fastapi.tiangolo.com/tutorial/query-params/), [SQLAlchemy SQL functions](https://docs.sqlalchemy.org/en/21/core/functions.html), [PostgreSQL date_trunc](https://www.postgresql.org/docs/13/functions-datetime.html).
