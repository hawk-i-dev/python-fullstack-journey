Day 15 is SQLAlchemy ORM. Until now you used raw SQL with `psycopg`. Today you learn the professional layer many FastAPI projects use to work with database tables as Python objects.

**Day 15 Goal**
You should understand:

- What ORM means
- How a database table becomes a Python class
- How FastAPI uses a database session
- Basic CRUD using SQLAlchemy
- Difference between raw SQL and ORM code

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-15-sqlalchemy-orm
cd day-15-sqlalchemy-orm
code .
```

Install:

```powershell
python -m pip install fastapi uvicorn sqlalchemy psycopg[binary] pydantic-settings
```

Create structure:

```text
day-15-sqlalchemy-orm/
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

`.env`:

```env
DB_NAME=python_fullstack_day15
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
```

In `psql`:

```sql
CREATE DATABASE python_fullstack_day15;
```

`app/config.py`:

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

`app/db.py`:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import settings

DATABASE_URL = (
    f"postgresql+psycopg://{settings.db_user}:{settings.db_password}"
    f"@{settings.db_host}:{settings.db_port}/{settings.db_name}"
)

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)


class Base(DeclarativeBase):
    pass


def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

`app/models.py`:

```python
from sqlalchemy import DateTime, Numeric, String, func
from sqlalchemy.orm import Mapped, mapped_column

from app.db import Base


class Expense(Base):
    __tablename__ = "expenses"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    title: Mapped[str] = mapped_column(String(100))
    amount: Mapped[float] = mapped_column(Numeric(10, 2))
    category: Mapped[str] = mapped_column(String(50))
    created_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())
```

`app/schemas.py`:

```python
from datetime import datetime

from pydantic import BaseModel, Field


class ExpenseCreate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category: str = Field(min_length=1)


class ExpenseResponse(BaseModel):
    id: int
    title: str
    amount: float
    category: str
    created_at: datetime

    model_config = {"from_attributes": True}
```

`app/main.py`:

```python
from fastapi import Depends, FastAPI, HTTPException
from sqlalchemy.orm import Session

from app.db import Base, engine, get_db
from app.models import Expense
from app.schemas import ExpenseCreate, ExpenseResponse

Base.metadata.create_all(bind=engine)

app = FastAPI(title="Expense API With SQLAlchemy")


@app.get("/health")
def health_check():
    return {"status": "ok"}


@app.post("/expenses", response_model=ExpenseResponse, status_code=201)
def create_expense(expense_data: ExpenseCreate, db: Session = Depends(get_db)):
    expense = Expense(
        title=expense_data.title,
        amount=expense_data.amount,
        category=expense_data.category,
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


@app.delete("/expenses/{expense_id}")
def delete_expense(expense_id: int, db: Session = Depends(get_db)):
    expense = db.query(Expense).filter(Expense.id == expense_id).first()

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    db.delete(expense)
    db.commit()

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

Test:

- `GET /health`
- `POST /expenses`
- `GET /expenses`
- `GET /expenses/1`
- `DELETE /expenses/1`

**Important Concept**
Raw SQL version:

```python
cursor.execute("SELECT * FROM expenses")
```

SQLAlchemy ORM version:

```python
db.query(Expense).all()
```

`Expense` is a Python class, but SQLAlchemy maps it to the `expenses` table.

**Challenge**
Add `PUT /expenses/{expense_id}` to update title, amount, and category.

Commit:

```powershell
git add .
git commit -m "Day 15 learn SQLAlchemy ORM"
```

Day 15 is complete when you can explain what `Base`, `Expense`, `SessionLocal`, `db.add()`, `db.commit()`, and `db.refresh()` do.
