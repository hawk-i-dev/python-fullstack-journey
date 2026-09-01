Day 57 = Expense filters, sorting, pagination, and summary API.

Today we make `/expenses` behave like a real production API.

## 1. Start backend

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

If server is already running, stop it with `Ctrl + C`, then restart after code changes.

## 2. Update `app/schemas.py`

Add this below your expense schemas:

```python
class ExpenseSummaryResponse(BaseModel):
    total_expenses: int
    total_amount: Decimal
    average_amount: Decimal
```

Make sure these imports exist at the top:

```python
from decimal import Decimal
from datetime import datetime
```

## 3. Replace `app/routers/expenses.py`

```python
from datetime import date, datetime, timedelta
from decimal import Decimal
from typing import Literal

from fastapi import APIRouter, Depends, HTTPException, Query, status
from sqlalchemy import func, select
from sqlalchemy.orm import Session, selectinload

from app.db import get_db
from app.models import Category, Expense, User
from app.schemas import ExpenseCreate, ExpenseResponse, ExpenseSummaryResponse, ExpenseUpdate
from app.security import get_current_user

router = APIRouter(prefix="/expenses", tags=["expenses"])


def get_owned_category(db: Session, category_id: int, user_id: int) -> Category:
    category = db.scalar(
        select(Category).where(
            Category.id == category_id,
            Category.user_id == user_id,
        )
    )

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    return category


def get_owned_expense(db: Session, expense_id: int, user_id: int) -> Expense:
    expense = db.scalar(
        select(Expense)
        .options(selectinload(Expense.category))
        .where(
            Expense.id == expense_id,
            Expense.user_id == user_id,
        )
    )

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    return expense


def apply_expense_filters(
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
        get_owned_category(db, category_id, user_id)
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


@router.post("", response_model=ExpenseResponse, status_code=status.HTTP_201_CREATED)
def create_expense(
    expense_data: ExpenseCreate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    title = expense_data.title.strip()

    if not title:
        raise HTTPException(status_code=400, detail="Title cannot be empty")

    get_owned_category(db, expense_data.category_id, current_user.id)

    expense = Expense(
        title=title,
        amount=expense_data.amount,
        category_id=expense_data.category_id,
        user_id=current_user.id,
    )

    db.add(expense)
    db.commit()
    db.refresh(expense)

    return get_owned_expense(db, expense.id, current_user.id)


@router.get("", response_model=list[ExpenseResponse])
def list_expenses(
    q: str | None = Query(default=None, max_length=100),
    category_id: int | None = Query(default=None, gt=0),
    start_date: date | None = None,
    end_date: date | None = None,
    min_amount: Decimal | None = Query(default=None, gt=0),
    max_amount: Decimal | None = Query(default=None, gt=0),
    sort_by: Literal["created_at", "amount", "title"] = "created_at",
    sort_order: Literal["asc", "desc"] = "desc",
    limit: int = Query(default=100, ge=1, le=500),
    offset: int = Query(default=0, ge=0),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    statement = select(Expense).options(selectinload(Expense.category))

    statement = apply_expense_filters(
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

    statement = statement.offset(offset).limit(limit)

    return db.scalars(statement).all()


@router.get("/summary", response_model=ExpenseSummaryResponse)
def get_expense_summary(
    q: str | None = Query(default=None, max_length=100),
    category_id: int | None = Query(default=None, gt=0),
    start_date: date | None = None,
    end_date: date | None = None,
    min_amount: Decimal | None = Query(default=None, gt=0),
    max_amount: Decimal | None = Query(default=None, gt=0),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    statement = select(
        func.count(Expense.id),
        func.coalesce(func.sum(Expense.amount), 0),
        func.coalesce(func.avg(Expense.amount), 0),
    )

    statement = apply_expense_filters(
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

    total_expenses, total_amount, average_amount = db.execute(statement).one()

    return {
        "total_expenses": total_expenses,
        "total_amount": total_amount,
        "average_amount": average_amount,
    }


@router.get("/{expense_id}", response_model=ExpenseResponse)
def get_expense(
    expense_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    return get_owned_expense(db, expense_id, current_user.id)


@router.put("/{expense_id}", response_model=ExpenseResponse)
def update_expense(
    expense_id: int,
    expense_data: ExpenseUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    expense = get_owned_expense(db, expense_id, current_user.id)

    if expense_data.title is not None:
        title = expense_data.title.strip()

        if not title:
            raise HTTPException(status_code=400, detail="Title cannot be empty")

        expense.title = title

    if expense_data.amount is not None:
        expense.amount = expense_data.amount

    if expense_data.category_id is not None:
        get_owned_category(db, expense_data.category_id, current_user.id)
        expense.category_id = expense_data.category_id

    db.commit()

    return get_owned_expense(db, expense.id, current_user.id)


@router.delete("/{expense_id}")
def delete_expense(
    expense_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    expense = get_owned_expense(db, expense_id, current_user.id)

    db.delete(expense)
    db.commit()

    return {"message": "Expense deleted successfully"}
```

Important: `/summary` must be above `/{expense_id}`. Otherwise FastAPI may treat `summary` as an `expense_id`.

## 4. Test in Swagger

Open:

```text
http://127.0.0.1:8000/docs
```

Test these after login and Authorize:

```text
GET /expenses?q=breakfast
GET /expenses?category_id=1
GET /expenses?start_date=2026-09-01&end_date=2026-09-30
GET /expenses?min_amount=100&max_amount=1000
GET /expenses?sort_by=amount&sort_order=desc
GET /expenses?limit=5&offset=0
GET /expenses/summary
```

## 5. Senior concept for today

A real list API is not just `GET all`.

It should support:

```text
filtering
searching
sorting
pagination
summary data
ownership security
```

Every query must still include:

```python
Expense.user_id == current_user.id
```

That is the protection line.

## 6. Commit

```powershell
git status
git add .
git commit -m "Day 57 add expense filters and summary API"
```

Sources checked: [FastAPI query parameters](https://fastapi.tiangolo.com/tutorial/query-params/), [FastAPI Query validation](https://fastapi.tiangolo.com/reference/parameters/), [SQLAlchemy select/order/group functions](https://docs.sqlalchemy.org/en/21/orm/queryguide/select.html).
