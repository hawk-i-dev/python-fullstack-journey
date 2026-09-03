Day 70 = Production readiness + portfolio polish.

Today we make the project presentable like a real portfolio repo.

Target:

```text
Config cleanup
Professional README
Architecture docs
Release checklist
Changelog
v0.1.0 tag
```

## 1. Make backend CORS configurable

Open:

```text
backend/app/config.py
```

Inside `Settings`, add:

```python
cors_origins: str = "http://localhost:5173,http://127.0.0.1:5173"

@property
def cors_origin_list(self) -> list[str]:
    return [origin.strip() for origin in self.cors_origins.split(",") if origin.strip()]
```

Now open:

```text
backend/app/main.py
```

Replace hardcoded frontend origins:

```python
frontend_origins = [
    "http://localhost:5173",
    "http://127.0.0.1:5173",
]
```

with:

```python
from app.config import settings
```

and use:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origin_list,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    allow_headers=["Authorization", "Content-Type"],
)
```

## 2. Update `backend/.env.example`

```env
DB_NAME=expense_manager
DB_USER=postgres
DB_PASSWORD=your_postgres_password
DB_HOST=localhost
DB_PORT=5432

JWT_SECRET_KEY=replace_with_secure_random_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

CORS_ORIGINS=http://localhost:5173,http://127.0.0.1:5173
```

## 3. Create root `README.md`

Create/replace:

```text
README.md
```

Use this:

```md
# Expense Manager SaaS

A full-stack expense tracking application built with FastAPI, PostgreSQL, React, TypeScript, and Docker-ready architecture.

![CI](https://github.com/YOUR_USERNAME/expense-manager-saas/actions/workflows/ci.yml/badge.svg)

## Features

- User registration and JWT login
- User-owned categories
- User-owned expenses
- Expense search, filters, sorting, and pagination
- Dashboard summary
- Category-wise reports
- Monthly reports
- CSV exports
- Backend API tests with pytest
- Frontend unit tests with Vitest
- End-to-end tests with Playwright
- GitHub Actions CI pipeline

## Tech Stack

### Backend

- Python
- FastAPI
- PostgreSQL
- SQLAlchemy
- Alembic
- PyJWT
- pytest

### Frontend

- React
- TypeScript
- Vite
- React Router
- Vitest
- Playwright

## Project Structure

```text
expense-manager-saas/
├── backend/
│   ├── app/
│   ├── alembic/
│   └── tests/
├── frontend/
│   ├── src/
│   └── e2e/
├── docs/
├── .github/workflows/
└── README.md
```

## Backend Setup

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Create PostgreSQL database:

```sql
CREATE DATABASE expense_manager;
```

Create `.env` from `.env.example`.

Run migrations:

```powershell
alembic upgrade head
```

Start backend:

```powershell
uvicorn app.main:app --reload
```

Backend runs at:

```text
http://127.0.0.1:8000
```

API docs:

```text
http://127.0.0.1:8000/docs
```

## Frontend Setup

```powershell
cd frontend
npm install
npm run dev
```

Frontend runs at:

```text
http://localhost:5173
```

## Test Commands

Backend:

```powershell
cd backend
pytest
```

Frontend unit tests:

```powershell
cd frontend
npm run test:run
```

Frontend E2E tests:

```powershell
cd frontend
npm run test:e2e
```

Frontend build:

```powershell
cd frontend
npm run build
```

## Main API Endpoints

### Auth

```text
POST /auth/register
POST /auth/login
GET  /auth/me
```

### Categories

```text
GET    /categories
POST   /categories
GET    /categories/{category_id}
PUT    /categories/{category_id}
DELETE /categories/{category_id}
```

### Expenses

```text
GET    /expenses
POST   /expenses
GET    /expenses/summary
GET    /expenses/{expense_id}
PUT    /expenses/{expense_id}
DELETE /expenses/{expense_id}
```

### Reports

```text
GET /reports/dashboard
GET /reports/category-summary
GET /reports/monthly-summary
```

### Exports

```text
GET /exports/expenses.csv
GET /exports/category-summary.csv
GET /exports/monthly-summary.csv
```

## Security Notes

- Passwords are hashed before storage.
- JWT is required for private APIs.
- Users can access only their own categories and expenses.
- CSV export queries are user-owned.
- Frontend tokens are stored in localStorage for learning purposes.

## Status

Version: `v0.1.0`

This project is under active development as a Python full-stack portfolio application.
```

Replace:

```text
YOUR_USERNAME
```

with your GitHub username.

## 4. Create `docs/architecture.md`

```md
# Architecture

## Overview

Expense Manager SaaS is a full-stack application with a React frontend, FastAPI backend, and PostgreSQL database.

## Request Flow

```text
Browser
  ↓
React frontend
  ↓ Authorization: Bearer token
FastAPI backend
  ↓
SQLAlchemy ORM
  ↓
PostgreSQL database
```

## Backend Layers

```text
routers/     API endpoints
schemas.py   request/response validation
models.py    database tables
security.py  password hashing and JWT handling
db.py        database connection/session
config.py    environment-based settings
```

## Ownership Rule

Every private data query must filter by the current logged-in user.

Example:

```python
Expense.user_id == current_user.id
Category.user_id == current_user.id
```

This prevents one user from reading or modifying another user's data.

## Testing Strategy

```text
pytest       backend API and business rules
Vitest       frontend components and API client
Playwright   real browser user journeys
GitHub CI    automated verification on push and pull request
```
```

## 5. Create `docs/release-checklist.md`

```md
# Release Checklist

Before creating a release tag:

## Backend

- [ ] `pytest` passes
- [ ] Alembic migrations run successfully
- [ ] `/health` works
- [ ] `/health/db` works
- [ ] Auth flow works
- [ ] User ownership checks work

## Frontend

- [ ] `npm run test:run` passes
- [ ] `npm run build` passes
- [ ] Login works
- [ ] Register works
- [ ] Categories CRUD works
- [ ] Expenses CRUD works
- [ ] Reports page works
- [ ] CSV exports work

## E2E

- [ ] `npm run test:e2e` passes

## GitHub

- [ ] CI passes
- [ ] README is updated
- [ ] CHANGELOG is updated
- [ ] No `.env` file committed
- [ ] No `.venv`, `node_modules`, or test reports committed
```

## 6. Create `CHANGELOG.md`

```md
# Changelog

All notable changes to this project will be documented in this file.

## [0.1.0] - 2026-09-03

### Added

- FastAPI backend foundation
- PostgreSQL database integration
- SQLAlchemy models for users, categories, and expenses
- Alembic migrations
- JWT authentication
- User-owned categories API
- User-owned expenses API
- Expense filtering, sorting, pagination, and summary
- Reports APIs for dashboard, category summary, and monthly summary
- CSV export APIs
- React TypeScript frontend
- Frontend authentication flow
- Categories UI
- Expenses UI
- Dashboard and reports UI
- CSV export UI
- Backend pytest test suite
- Frontend Vitest test suite
- Playwright end-to-end tests
- GitHub Actions CI pipeline
```

## 7. Run all checks

Backend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
pytest
```

Frontend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run test:run
npm run build
npm run test:e2e
```

## 8. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 70 prepare project for portfolio release"
git push
```

## 9. Create release tag

Only do this after GitHub Actions passes.

```powershell
git tag -a v0.1.0 -m "Initial portfolio release"
git push origin v0.1.0
```

Senior concept: a portfolio project is not only code. A serious repo explains what it does, how to run it, how it is tested, how it is secured, and what version is released.

Sources checked: [GitHub README docs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes), [Keep a Changelog](https://keepachangelog.com/en/2.0.0/), [Semantic Versioning](https://semver.org/), [Git tag docs](https://git-scm.com/docs/git-tag.html).
