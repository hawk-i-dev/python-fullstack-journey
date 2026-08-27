Regular `Day 52`: build the real FastAPI backend foundation inside `expense-manager-saas/backend`.

Sources checked: FastAPI recommends splitting bigger apps into multiple files with package structure and routers; SQLAlchemy uses `create_engine`, `sessionmaker`, and `Session` for database work. References: [FastAPI bigger applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/), [SQLAlchemy session basics](https://docs.sqlalchemy.org/en/20/orm/session_basics.html), [Alembic tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html).

**Goal**

Create a backend that can:

- Start FastAPI
- Read `.env`
- Connect to PostgreSQL config
- Expose `/health`
- Expose `/health/db`
- Prepare structure for auth, categories, expenses, tests, and migrations

**Commands**

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install fastapi uvicorn sqlalchemy psycopg[binary] pydantic-settings alembic python-multipart pyjwt pwdlib[argon2] pytest httpx
```

Create this structure:

```text
backend/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   └── db.py
├── tests/
│   └── __init__.py
├── .env
├── .env.example
└── requirements.txt
```

**Files**

`.env`:

```env
DB_NAME=expense_manager
DB_USER=postgres
DB_PASSWORD=your_postgres_password
DB_HOST=localhost
DB_PORT=5432
JWT_SECRET_KEY=change_this_to_a_long_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

`.env.example`:

```env
DB_NAME=expense_manager
DB_USER=postgres
DB_PASSWORD=change_me
DB_HOST=localhost
DB_PORT=5432
JWT_SECRET_KEY=change_me_to_a_long_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

`requirements.txt`:

```text
fastapi
uvicorn
sqlalchemy
psycopg[binary]
pydantic-settings
alembic
python-multipart
pyjwt
pwdlib[argon2]
pytest
httpx
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
    jwt_secret_key: str
    jwt_algorithm: str
    access_token_expire_minutes: int

    model_config = SettingsConfigDict(env_file=".env")


settings = Settings()
```

`app/db.py`:

```python
from sqlalchemy import create_engine, text
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


def check_database_connection():
    with engine.connect() as connection:
        connection.execute(text("SELECT 1"))
```

`app/main.py`:

```python
from fastapi import FastAPI, HTTPException

from app.db import check_database_connection

app = FastAPI(title="Expense Manager SaaS API")


@app.get("/health")
def health_check():
    return {"status": "ok"}


@app.get("/health/db")
def database_health_check():
    try:
        check_database_connection()
        return {"database": "connected"}
    except Exception:
        raise HTTPException(status_code=500, detail="Database connection failed")
```

**Create PostgreSQL Database**

In `psql`:

```sql
CREATE DATABASE expense_manager;
```

**Run Backend**

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test:

- `GET /health`
- `GET /health/db`

If `/health` works but `/health/db` fails, your FastAPI setup is fine but PostgreSQL config/password/database is wrong.

**Save Dependencies**

```powershell
python -m pip freeze > requirements.txt
```

**Commit**

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"
git status
git add .
git commit -m "Day 52 create FastAPI backend foundation"
```

Day 52 is complete when `/health` and `/health/db` both return success. Day 53 will add SQLAlchemy models and Alembic migrations.
