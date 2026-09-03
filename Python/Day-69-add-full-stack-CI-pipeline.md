Day 69 = GitHub Actions CI for full-stack project.

Today we automate checks on GitHub:

```text
Backend pytest + PostgreSQL
Frontend Vitest
Frontend build
Playwright E2E with real backend + database
```

## 1. Update backend test DB config

Open:

```text
backend/tests/conftest.py
```

Add import:

```python
import os
```

Replace hardcoded test DB URL with:

```python
TEST_DATABASE_URL = os.getenv(
    "TEST_DATABASE_URL",
    "postgresql+psycopg://postgres:your_postgres_password@localhost:5432/expense_manager_test",
)
```

Local tests still work. CI can inject its own DB URL.

## 2. Update backend requirements

From backend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
python -m pip freeze > requirements.txt
```

## 3. Create GitHub Actions workflow

Create:

```text
.github/workflows/ci.yml
```

Add:

```yaml
name: CI

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

permissions:
  contents: read

jobs:
  backend:
    name: Backend tests
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: expense_manager_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    env:
      DB_NAME: expense_manager_test
      DB_USER: postgres
      DB_PASSWORD: postgres
      DB_HOST: localhost
      DB_PORT: 5432
      TEST_DATABASE_URL: postgresql+psycopg://postgres:postgres@localhost:5432/expense_manager_test
      JWT_SECRET_KEY: test-secret-key-for-ci-only
      JWT_ALGORITHM: HS256
      ACCESS_TOKEN_EXPIRE_MINUTES: 30

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Set up Python
        uses: actions/setup-python@v6
        with:
          python-version: "3.13"
          cache: pip
          cache-dependency-path: backend/requirements.txt

      - name: Install backend dependencies
        working-directory: backend
        run: |
          python -m pip install --upgrade pip
          python -m pip install -r requirements.txt

      - name: Run migrations
        working-directory: backend
        run: alembic upgrade head

      - name: Run backend tests
        working-directory: backend
        run: pytest -v

  frontend:
    name: Frontend tests and build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Set up Node
        uses: actions/setup-node@v7
        with:
          node-version: "lts/*"
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - name: Install frontend dependencies
        working-directory: frontend
        run: npm ci

      - name: Run frontend unit tests
        working-directory: frontend
        run: npm run test:run

      - name: Build frontend
        working-directory: frontend
        run: npm run build

  e2e:
    name: Playwright E2E
    runs-on: ubuntu-latest
    needs: [backend, frontend]

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: expense_manager_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    env:
      DB_NAME: expense_manager_test
      DB_USER: postgres
      DB_PASSWORD: postgres
      DB_HOST: localhost
      DB_PORT: 5432
      TEST_DATABASE_URL: postgresql+psycopg://postgres:postgres@localhost:5432/expense_manager_test
      JWT_SECRET_KEY: test-secret-key-for-ci-only
      JWT_ALGORITHM: HS256
      ACCESS_TOKEN_EXPIRE_MINUTES: 30
      VITE_API_BASE_URL: http://127.0.0.1:8000

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Set up Python
        uses: actions/setup-python@v6
        with:
          python-version: "3.13"
          cache: pip
          cache-dependency-path: backend/requirements.txt

      - name: Install backend dependencies
        working-directory: backend
        run: |
          python -m pip install --upgrade pip
          python -m pip install -r requirements.txt

      - name: Run migrations
        working-directory: backend
        run: alembic upgrade head

      - name: Start backend
        working-directory: backend
        run: |
          python -m uvicorn app.main:app --host 127.0.0.1 --port 8000 &
          sleep 5
          curl --fail http://127.0.0.1:8000/health/db

      - name: Set up Node
        uses: actions/setup-node@v7
        with:
          node-version: "lts/*"
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - name: Install frontend dependencies
        working-directory: frontend
        run: npm ci

      - name: Install Playwright browser
        working-directory: frontend
        run: npx playwright install --with-deps chromium

      - name: Run Playwright tests
        working-directory: frontend
        run: npm run test:e2e

      - name: Upload Playwright report
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v5
        with:
          name: playwright-report
          path: frontend/playwright-report/
          retention-days: 30
```

## 4. Local check before commit

Run backend tests:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
pytest
```

Run frontend checks:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run test:run
npm run build
```

Run E2E:

```powershell
npm run test:e2e
```

## 5. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 69 add full stack CI pipeline"
git push
```

Then open GitHub repo → `Actions` tab.

Expected:

```text
Backend tests passed
Frontend tests and build passed
Playwright E2E passed
```

Senior concept: CI is your safety gate. If tests pass only on your laptop, that is not enough. A portfolio project looks stronger when GitHub can prove the backend, frontend, database, and browser flow work from a clean machine.

Sources checked: [GitHub PostgreSQL service containers](https://docs.github.com/en/actions/tutorials/use-containerized-services/create-postgresql-service-containers), [GitHub Python CI](https://docs.github.com/en/actions/tutorials/build-and-test-code/python), [setup-node caching](https://github.com/actions/setup-node/blob/main/README.md), [Playwright CI](https://playwright.dev/docs/ci).
