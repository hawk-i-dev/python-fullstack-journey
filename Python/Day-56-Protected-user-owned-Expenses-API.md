Day 56 = Protected user-owned Expenses API.

Goal: logged-in user can create, list, view, update, and delete only their own expenses. Expense must use a category owned by the same user.

## 1. Go to backend

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
```

## 2. Small professional fix in `app/models.py`

For money, use `Decimal`, not `float`.

Add this import:

```python
from decimal import Decimal
```

Update the expense amount line:

```python
amount: Mapped[Decimal] = mapped_column(Numeric(10, 2))
```

No Alembic migration is required for this if the column is already `Numeric(10, 2)`.

## 3. Update `app/schemas.py`

Add this below your category schemas:

```python
from datetime import datetime
from decimal import Decimal

class ExpenseCreate(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    amount: Decimal = Field(gt=0, max_digits=10, decimal_places=2)
    category_id: int = Field(gt=0)


class ExpenseUpdate(BaseModel):
    title: str | None = Field(default=None, min_length=1, max_length=100)
    amount: Decimal | None = Field(default=None, gt=0, max_digits=10, decimal_places=2)
    category_id: int | None = Field(default=None, gt=0)


class ExpenseResponse(BaseModel):
    id: int
    title: str
    amount: Decimal
    category_id: int
    user_id: int
    created_at: datetime
    category: CategoryResponse | None = None

    model_config = {"from_attributes": True}
```

## 4. Create `app/routers/expenses.py`

```python
from fastapi import APIRouter, Depends, HTTPException, Query, status
from sqlalchemy import select
from sqlalchemy.orm import Session, selectinload

from app.db import get_db
from app.models import Category, Expense, User
from app.schemas import ExpenseCreate, ExpenseResponse, ExpenseUpdate
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

    return expense


@router.get("", response_model=list[ExpenseResponse])
def list_expenses(
    category_id: int | None = Query(default=None, gt=0),
    limit: int = Query(default=100, ge=1, le=500),
    offset: int = Query(default=0, ge=0),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    statement = (
        select(Expense)
        .options(selectinload(Expense.category))
        .where(Expense.user_id == current_user.id)
        .order_by(Expense.created_at.desc())
        .offset(offset)
        .limit(limit)
    )

    if category_id is not None:
        statement = statement.where(Expense.category_id == category_id)

    return db.scalars(statement).all()


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
    db.refresh(expense)

    return expense


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

## 5. Update `app/main.py`

```python
from app.routers import auth, categories, expenses

app.include_router(auth.router)
app.include_router(categories.router)
app.include_router(expenses.router)
```

## 6. Run server

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

## 7. Test order in Swagger

First authorize:

1. `POST /auth/register`
2. `POST /auth/login`
3. Click `Authorize`
4. `POST /categories`

Example category:

```json
{
  "name": "Food"
}
```

Then create expense:

```json
{
  "title": "Breakfast",
  "amount": "120.50",
  "category_id": 1
}
```

Test these:

```text
POST   /expenses
GET    /expenses
GET    /expenses/{expense_id}
PUT    /expenses/{expense_id}
DELETE /expenses/{expense_id}
```

## Key senior-level point

Never trust `category_id` directly. Always check:

```python
Category.id == category_id
Category.user_id == current_user.id
```

Otherwise one user could create expenses under another user’s category.

## Commit

```powershell
git status
git add .
git commit -m "Day 56 add protected user owned expenses API"
```

Sources checked: [FastAPI routers](https://fastapi.tiangolo.com/tutorial/bigger-applications/), [SQLAlchemy ORM select](https://docs.sqlalchemy.org/en/21/orm/queryguide/select.html), [Pydantic Decimal constraints](https://pydantic.dev/docs/validation/2.12/api/pydantic/standard_library_types/).
