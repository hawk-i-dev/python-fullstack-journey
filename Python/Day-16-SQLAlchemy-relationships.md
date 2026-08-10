Day 16 is SQLAlchemy relationships. Today you convert the single `expenses` table into a proper relational model: `categories` and `expenses`.

**Day 16 Goal**
You should understand:

- `ForeignKey`
- One-to-many relationship
- `relationship()`
- Creating related records
- Returning nested data from FastAPI
- Why `category_id` is better than repeated category text

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-16-sqlalchemy-relationships
cd day-16-sqlalchemy-relationships
code .
```

Install:

```powershell
python -m pip install fastapi uvicorn sqlalchemy psycopg[binary] pydantic-settings
```

Create database:

```sql
CREATE DATABASE python_fullstack_day16;
```

Use the same structure as Day 15:

```text
day-16-sqlalchemy-relationships/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── db.py
│   ├── models.py
│   └── schemas.py
├── .env
├── .gitignore
└── requirements.txt
```

Use the same `config.py` and `db.py` from Day 15, but set `.env` to:

```env
DB_NAME=python_fullstack_day16
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
```

`app/models.py`:

```python
from sqlalchemy import DateTime, ForeignKey, Numeric, String, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db import Base


class Category(Base):
    __tablename__ = "categories"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    name: Mapped[str] = mapped_column(String(50), unique=True)

    expenses: Mapped[list["Expense"]] = relationship(back_populates="category")


class Expense(Base):
    __tablename__ = "expenses"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    title: Mapped[str] = mapped_column(String(100))
    amount: Mapped[float] = mapped_column(Numeric(10, 2))
    category_id: Mapped[int] = mapped_column(ForeignKey("categories.id"))
    created_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())

    category: Mapped[Category] = relationship(back_populates="expenses")
```

`app/schemas.py`:

```python
from datetime import datetime

from pydantic import BaseModel, Field


class CategoryCreate(BaseModel):
    name: str = Field(min_length=1)


class CategoryResponse(BaseModel):
    id: int
    name: str

    model_config = {"from_attributes": True}


class ExpenseCreate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category_id: int


class ExpenseResponse(BaseModel):
    id: int
    title: str
    amount: float
    created_at: datetime
    category: CategoryResponse

    model_config = {"from_attributes": True}
```

`app/main.py`:

```python
from fastapi import Depends, FastAPI, HTTPException
from sqlalchemy.orm import Session

from app.db import Base, engine, get_db
from app.models import Category, Expense
from app.schemas import CategoryCreate, CategoryResponse, ExpenseCreate, ExpenseResponse

Base.metadata.create_all(bind=engine)

app = FastAPI(title="Expense API With Relationships")


@app.get("/health")
def health_check():
    return {"status": "ok"}


@app.post("/categories", response_model=CategoryResponse, status_code=201)
def create_category(category_data: CategoryCreate, db: Session = Depends(get_db)):
    category = Category(name=category_data.name.strip())

    db.add(category)
    db.commit()
    db.refresh(category)

    return category


@app.get("/categories", response_model=list[CategoryResponse])
def list_categories(db: Session = Depends(get_db)):
    return db.query(Category).order_by(Category.id).all()


@app.post("/expenses", response_model=ExpenseResponse, status_code=201)
def create_expense(expense_data: ExpenseCreate, db: Session = Depends(get_db)):
    category = db.query(Category).filter(Category.id == expense_data.category_id).first()

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    expense = Expense(
        title=expense_data.title.strip(),
        amount=expense_data.amount,
        category_id=expense_data.category_id,
    )

    db.add(expense)
    db.commit()
    db.refresh(expense)

    return expense


@app.get("/expenses", response_model=list[ExpenseResponse])
def list_expenses(db: Session = Depends(get_db)):
    return db.query(Expense).order_by(Expense.id).all()


@app.get("/expenses/{expense_id}", response_model=ExpenseResponse)
def get_expense(expense_id: int, db: Session = Depends(get_db)):
    expense = db.query(Expense).filter(Expense.id == expense_id).first()

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    return expense
```

Run:

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test in this order:

1. `POST /categories`

```json
{
  "name": "Food"
}
```

2. `POST /categories`

```json
{
  "name": "Travel"
}
```

3. `POST /expenses`

```json
{
  "title": "Tea",
  "amount": 20,
  "category_id": 1
}
```

4. `GET /expenses`

You should see the expense with nested category data.

**Challenge**
Add this endpoint:

```text
GET /categories/{category_id}/expenses
```

It should return all expenses for one category.

Commit:

```powershell
git add .
git commit -m "Day 16 learn SQLAlchemy relationships"
```

Day 16 is complete when you can explain this: one category can have many expenses, but each expense belongs to one category.
