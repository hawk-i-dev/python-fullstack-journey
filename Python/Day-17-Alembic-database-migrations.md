Day 17 is Alembic database migrations. This is a professional backend skill. Until now we used:

```python
Base.metadata.create_all(bind=engine)
```

That is okay for learning, but real projects use migrations to track database changes safely.

**Day 17 Goal**
You should understand:

- What a migration is
- Why `create_all()` is not enough for real projects
- How Alembic tracks database changes
- How to create and apply migrations
- How models and database schema stay connected

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-17-alembic-migrations
cd day-17-alembic-migrations
code .
```

Install:

```powershell
python -m pip install fastapi uvicorn sqlalchemy psycopg[binary] pydantic-settings alembic
```

Create database:

```sql
CREATE DATABASE python_fullstack_day17;
```

Create project files like Day 16:

```text
day-17-alembic-migrations/
├── app/
│   ├── __init__.py
│   ├── config.py
│   ├── db.py
│   ├── models.py
│   └── main.py
├── .env
├── .gitignore
└── requirements.txt
```

Use `.env`:

```env
DB_NAME=python_fullstack_day17
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
```

Use the same `config.py`, `db.py`, and `models.py` from Day 16.

In `app/main.py`, remove this line if you have it:

```python
Base.metadata.create_all(bind=engine)
```

Reason: Alembic will create/update tables now.

Initialize Alembic:

```powershell
alembic init alembic
```

This creates:

```text
alembic/
├── env.py
├── script.py.mako
└── versions/
alembic.ini
```

Open `alembic/env.py`.

Find this line:

```python
target_metadata = None
```

Replace it with:

```python
from app.db import Base
from app.models import Category, Expense

target_metadata = Base.metadata
```

Now open `alembic.ini`.

Find:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Replace it with your database URL:

```ini
sqlalchemy.url = postgresql+psycopg://postgres:your_password_here@localhost:5432/python_fullstack_day17
```

Create your first migration:

```powershell
alembic revision --autogenerate -m "create categories and expenses tables"
```

Apply it:

```powershell
alembic upgrade head
```

Check in `psql`:

```sql
\c python_fullstack_day17
\dt
```

You should see:

```text
alembic_version
categories
expenses
```

Run your API:

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
- `GET /categories`
- `POST /expenses`
- `GET /expenses`

**Challenge**
Add a new column to `Expense`:

```python
description: Mapped[str | None] = mapped_column(String(255), nullable=True)
```

Then generate a second migration:

```powershell
alembic revision --autogenerate -m "add description to expenses"
alembic upgrade head
```

Day 17 is complete when you can explain this clearly:

`models.py` defines the intended table structure, Alembic compares it with PostgreSQL, creates a migration file, and `alembic upgrade head` applies that change to the database.
