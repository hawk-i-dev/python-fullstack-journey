Regular `Day 55`: protected user-owned Categories API.

Today you add categories to the real portfolio backend. Design correction: categories should belong to a user. In a SaaS app, two users should be able to create their own `Food` category without conflicting.

References checked: [FastAPI bigger applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/), [SQLAlchemy relationships](https://docs.sqlalchemy.org/en/20/tutorial/orm_related_objects.html), [SQLAlchemy unique constraints](https://docs.sqlalchemy.org/en/20/core/constraints.html), [Alembic tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html).

**Start**

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
```

Make sure this exists:

```text
app/routers/__init__.py
```

**Update `app/models.py`**

Replace with:

```python
from datetime import datetime

from sqlalchemy import Boolean, DateTime, ForeignKey, Numeric, String, UniqueConstraint, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    email: Mapped[str] = mapped_column(String(100), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)

    categories: Mapped[list["Category"]] = relationship(back_populates="owner")
    expenses: Mapped[list["Expense"]] = relationship(back_populates="owner")


class Category(Base):
    __tablename__ = "categories"
    __table_args__ = (
        UniqueConstraint("user_id", "name", name="uq_categories_user_id_name"),
    )

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    name: Mapped[str] = mapped_column(String(50), index=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)

    owner: Mapped[User] = relationship(back_populates="categories")
    expenses: Mapped[list["Expense"]] = relationship(back_populates="category")


class Expense(Base):
    __tablename__ = "expenses"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    title: Mapped[str] = mapped_column(String(100))
    amount: Mapped[float] = mapped_column(Numeric(10, 2))
    category_id: Mapped[int] = mapped_column(ForeignKey("categories.id"))
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)
    created_at: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())

    category: Mapped[Category] = relationship(back_populates="expenses")
    owner: Mapped[User] = relationship(back_populates="expenses")
```

**Update `app/schemas.py`**

Add:

```python
class CategoryCreate(BaseModel):
    name: str = Field(min_length=1, max_length=50)


class CategoryUpdate(BaseModel):
    name: str = Field(min_length=1, max_length=50)


class CategoryResponse(BaseModel):
    id: int
    name: str

    model_config = {"from_attributes": True}
```

**Create `app/routers/categories.py`**

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.exc import IntegrityError
from sqlalchemy.orm import Session

from app.db import get_db
from app.models import Category, User
from app.schemas import CategoryCreate, CategoryResponse, CategoryUpdate
from app.security import get_current_user

router = APIRouter(prefix="/categories", tags=["categories"])


@router.post("", response_model=CategoryResponse, status_code=201)
def create_category(
    category_data: CategoryCreate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    name = category_data.name.strip()

    if not name:
        raise HTTPException(status_code=400, detail="Category name is required")

    category = Category(name=name, user_id=current_user.id)

    try:
        db.add(category)
        db.commit()
        db.refresh(category)
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=400, detail="Category already exists")

    return category


@router.get("", response_model=list[CategoryResponse])
def list_categories(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    return (
        db.query(Category)
        .filter(Category.user_id == current_user.id)
        .order_by(Category.name)
        .all()
    )


@router.get("/{category_id}", response_model=CategoryResponse)
def get_category(
    category_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    category = (
        db.query(Category)
        .filter(Category.id == category_id, Category.user_id == current_user.id)
        .first()
    )

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    return category


@router.put("/{category_id}", response_model=CategoryResponse)
def update_category(
    category_id: int,
    category_data: CategoryUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    category = (
        db.query(Category)
        .filter(Category.id == category_id, Category.user_id == current_user.id)
        .first()
    )

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    category.name = category_data.name.strip()

    try:
        db.commit()
        db.refresh(category)
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=400, detail="Category already exists")

    return category


@router.delete("/{category_id}")
def delete_category(
    category_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    category = (
        db.query(Category)
        .filter(Category.id == category_id, Category.user_id == current_user.id)
        .first()
    )

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    try:
        db.delete(category)
        db.commit()
    except IntegrityError:
        db.rollback()
        raise HTTPException(
            status_code=400,
            detail="Cannot delete category with existing expenses",
        )

    return {"message": "Category deleted"}
```

**Update `app/main.py`**

```python
from app.routers import auth, categories

app.include_router(auth.router)
app.include_router(categories.router)
```

Keep your existing `/health` and `/health/db`.

**Migration**

```powershell
alembic revision --autogenerate -m "make categories user owned"
alembic upgrade head
```

Verify in `psql`:

```sql
\c expense_manager
\d categories
```

`categories` must include:

```text
user_id
uq_categories_user_id_name
```

**Test in Swagger**

Run:

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test order:

1. `POST /auth/register`
2. `POST /auth/login`
3. Authorize
4. `POST /categories`
5. `GET /categories`
6. `GET /categories/{category_id}`
7. `PUT /categories/{category_id}`
8. `DELETE /categories/{category_id}`

Commit:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"
git add .
git commit -m "Day 55 add protected user owned categories API"
```

Day 55 is complete when one logged-in user sees only their own categories.
