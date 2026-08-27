Regular `Day 51`: portfolio project kickoff.

From now on, stop creating isolated daily apps. Build one serious project daily.

Project name:

```text
expense-manager-saas
```

**Day 51 Goal**

Set up the real project structure and write the project plan before coding.

**Stack**

```text
Frontend: React + Vite
Backend: FastAPI
Database: PostgreSQL
ORM: SQLAlchemy
Migrations: Alembic
Auth: JWT
Testing: Pytest, Vitest, Playwright
Deployment: Docker Compose
CI: GitHub Actions
```

**Create Project**

```powershell
cd "$env:USERPROFILE\Documents"
mkdir expense-manager-saas
cd expense-manager-saas
git init
mkdir backend
mkdir frontend
mkdir docs
mkdir .github
mkdir .github\workflows
code .
```

Create `README.md`:

```md
# Expense Manager SaaS

A full-stack expense management app built with React, FastAPI, PostgreSQL, Docker, and CI.

## Features

- User registration and login
- JWT authentication
- User-owned expenses
- Categories
- Search, filter, sort, and pagination
- Reports dashboard
- Charts
- Automated tests
- Dockerized full-stack setup

## Tech Stack

- React
- FastAPI
- PostgreSQL
- SQLAlchemy
- Alembic
- Docker Compose
- GitHub Actions
```

Create `docs/requirements.md`:

```md
# Requirements

## Users

A user can register, login, logout, and view only their own data.

## Categories

A user can create and view categories.

## Expenses

A user can create, view, update, delete, search, filter, sort, and paginate expenses.

## Reports

A user can view total expense, expense count, and category-wise totals.

## Security

Passwords must be hashed.
JWT tokens must protect private routes.
Users must never access another user's expenses.
```

Create `docs/api-plan.md`:

```md
# API Plan

## Auth

POST /auth/register
POST /auth/login
GET /auth/me

## Categories

POST /categories
GET /categories

## Expenses

POST /expenses
GET /expenses
GET /expenses/{expense_id}
PUT /expenses/{expense_id}
DELETE /expenses/{expense_id}

## Reports

GET /expenses/summary
GET /expenses/reports/by-category
```

Create `.gitignore`:

```gitignore
.env
.env.*
!.env.example

.venv/
__pycache__/
.pytest_cache/
*.pyc

node_modules/
dist/
coverage/
playwright-report/
test-results/

.DS_Store
```

Create root `.env.example`:

```env
POSTGRES_DB=expense_manager
POSTGRES_USER=expense_user
POSTGRES_PASSWORD=change_me
JWT_SECRET_KEY=change_me_to_long_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
FRONTEND_PORT=3000
API_PORT=8000
DB_PORT=5433
```

**Commit**

```powershell
git status
git add .
git commit -m "Day 51 initialize portfolio project"
```

**Day 51 Completion Check**

You should be able to explain:

```text
This is no longer practice code. This is one portfolio project with documentation, planned API routes, planned features, and clean repo structure.
```

Next: `Day 52` will create the real FastAPI backend foundation inside `backend/`.
