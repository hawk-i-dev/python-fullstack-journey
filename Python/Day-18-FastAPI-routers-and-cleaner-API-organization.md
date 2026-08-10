Day 18 is FastAPI routers and cleaner API organization. Today you stop keeping all endpoints inside `main.py`.

**Day 18 Goal**
You should understand:

- `APIRouter`
- Splitting routes by feature
- Keeping `main.py` small
- Using route prefixes and tags
- Handling duplicate data properly
- Building APIs like a real backend project

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-18-fastapi-routers
cd day-18-fastapi-routers
code .
```

Use the Day 17 project style, but structure it like this:

```text
day-18-fastapi-routers/
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
├── alembic/
├── alembic.ini
├── .env
├── .gitignore
└── requirements.txt
```

Install if needed:

```powershell
python -m pip install fastapi uvicorn sqlalchemy psycopg[binary] pydantic-settings alembic
```

Create database:

```sql
CREATE DATABASE python_fullstack_day18;
```

Use `.env`:

```env
DB_NAME=python_fullstack_day18
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
```

Use the same `config.py`, `db.py`, `models.py`, and Alembic setup from Day 17.

Now create `app/routers/categories.py`:

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.exc import IntegrityError
from sqlalchemy.orm import Session

from app.db import get_db
from app.models import Category, Expense
from app.schemas import CategoryCreate, CategoryResponse, ExpenseResponse

router = APIRouter(prefix="/categories", tags=["categories"])


@router.post("", response_model=CategoryResponse, status_code=201)
def create_category(category_data: CategoryCreate, db: Session = Depends(get_db)):
    category = Category(name=category_data.name.strip())

    try:
        db.add(category)
        db.commit()
        db.refresh(category)
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=400, detail="Category already exists")

    return category


@router.get("", response_model=list[CategoryResponse])
def list_categories(db: Session = Depends(get_db)):
    return db.query(Category).order_by(Category.id).all()


@router.get("/{category_id}/expenses", response_model=list[ExpenseResponse])
def list_category_expenses(category_id: int, db: Session = Depends(get_db)):
    category = db.query(Category).filter(Category.id == category_id).first()

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    return db.query(Expense).filter(Expense.category_id == category_id).all()
```

Create `app/routers/expenses.py`:

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

from app.db import get_db
from app.models import Category, Expense
from app.schemas import ExpenseCreate, ExpenseResponse

router = APIRouter(prefix="/expenses", tags=["expenses"])


@router.post("", response_model=ExpenseResponse, status_code=201)
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


@router.get("", response_model=list[ExpenseResponse])
def list_expenses(db: Session = Depends(get_db)):
    return db.query(Expense).order_by(Expense.id).all()


@router.get("/{expense_id}", response_model=ExpenseResponse)
def get_expense(expense_id: int, db: Session = Depends(get_db)):
    expense = db.query(Expense).filter(Expense.id == expense_id).first()

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    return expense


@router.delete("/{expense_id}")
def delete_expense(expense_id: int, db: Session = Depends(get_db)):
    expense = db.query(Expense).filter(Expense.id == expense_id).first()

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    db.delete(expense)
    db.commit()

    return {"message": "Expense deleted"}
```

Now `app/main.py` becomes small:

```python
from fastapi import FastAPI

from app.routers import categories, expenses

app = FastAPI(title="Expense API With Routers")

app.include_router(categories.router)
app.include_router(expenses.router)


@app.get("/health")
def health_check():
    return {"status": "ok"}
```

Run migrations:

```powershell
alembic revision --autogenerate -m "create categories and expenses tables"
alembic upgrade head
```

Run API:

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test:

- `GET /health`
- `POST /categories`
- `POST /categories` again with same name
- `GET /categories`
- `POST /expenses`
- `GET /expenses`
- `GET /categories/1/expenses`
- `DELETE /expenses/1`

**Challenge**
Add `PUT /expenses/{expense_id}` inside `app/routers/expenses.py`.

It should update:

- `title`
- `amount`
- `category_id`

Before updating, validate that the new `category_id` exists.

Commit:

```powershell
git add .
git commit -m "Day 18 organize FastAPI app with routers"
```

Day 18 is complete when you can explain why `main.py` should only create the app and attach routers, while each router file owns endpoints for one feature.
