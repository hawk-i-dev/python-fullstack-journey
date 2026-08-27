Regular `Day 53`: SQLAlchemy models + Alembic migrations for the real project.

Today you create the actual database tables for the portfolio app.

**Day 53 Goal**

Create database models for:

- Users
- Categories
- Expenses

Then generate and apply Alembic migration.

**Start**

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
```

Install/check Alembic:

```powershell
python -m pip install alembic
```

Initialize Alembic if not already done:

```powershell
alembic init alembic
```

This creates:

```text
backend/
├── alembic/
│   ├── env.py
│   └── versions/
└── alembic.ini
```

**Create `app/models.py`**

```python
from sqlalchemy import Boolean, DateTime, ForeignKey, Numeric, String, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    email: Mapped[str] = mapped_column(String(100), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)

    expenses: Mapped[list["Expense"]] = relationship(back_populates="owner")


class Category(Base):
    __tablename__ = "categories"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    name: Mapped[str] = mapped_column(String(50), unique=True, index=True)

    expenses: Mapped[list["Expense"]] = relationship(back_populates="category")


class Expense(Base):
    __tablename__ = "expenses"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    title: Mapped[str] = mapped_column(String(100))
    amount: Mapped[float] = mapped_column(Numeric(10, 2))
    category_id: Mapped[int] = mapped_column(ForeignKey("categories.id"))
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)
    created_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())

    category: Mapped[Category] = relationship(back_populates="expenses")
    owner: Mapped[User] = relationship(back_populates="expenses")
```

**Update `alembic/env.py`**

Find:

```python
target_metadata = None
```

Replace with:

```python
from app.db import Base
from app.models import Category, Expense, User

target_metadata = Base.metadata
```

Also make sure imports can find `app`. If Alembic gives `ModuleNotFoundError: No module named 'app'`, add this near the top of `alembic/env.py`:

```python
from pathlib import Path
import sys

sys.path.append(str(Path(__file__).resolve().parents[1]))
```

**Update `alembic.ini`**

Find:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Replace it with:

```ini
sqlalchemy.url = postgresql+psycopg://postgres:your_postgres_password@localhost:5432/expense_manager
```

Use your real PostgreSQL password.

**Create Migration**

```powershell
alembic revision --autogenerate -m "create users categories and expenses tables"
```

Open the generated migration file in `alembic/versions/`.

Check it includes:

```text
create_table('users'
create_table('categories'
create_table('expenses'
```

Apply migration:

```powershell
alembic upgrade head
```

**Verify in PostgreSQL**

```sql
\c expense_manager
\dt
```

Expected:

```text
alembic_version
categories
expenses
users
```

Check columns:

```sql
\d users
\d categories
\d expenses
```

**Important Concept**

`models.py` is the intended database design.

Alembic compares:

```text
SQLAlchemy models vs current PostgreSQL schema
```

Then it creates a migration file.

`alembic upgrade head` applies that migration to the database.

**Commit**

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"
git add .
git commit -m "Day 53 add database models and migrations"
```

Day 53 is complete when PostgreSQL has `users`, `categories`, and `expenses` tables created by Alembic.
