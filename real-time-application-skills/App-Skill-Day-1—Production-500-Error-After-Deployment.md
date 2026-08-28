## App Skill Day 1 — Production 500 Error After Deployment

### Scenario

Deployment completed. After that, one API starts failing:

```text
POST /auth/login
GET /expenses
POST /expenses
```

Frontend or Swagger shows:

```text
500 Internal Server Error
```

This means:

```text
Server accepted the request, but backend code crashed while processing it.
```

Important: 500 is usually not a frontend issue. First check backend logs.

## Problem

Users cannot use one or more features after deployment.

Example:

```text
Login is failing after deployment.
Expense list is not loading.
Create expense returns 500.
```

## Impact

Ask this first:

```text
Is the full application down, or only one feature?
```

| Case | Meaning |
|---|---|
| `/health` fails | Full backend/server issue |
| only `/auth/login` fails | Auth logic/config/database issue |
| only `/expenses` fails | Expense code/schema/authorization issue |
| frontend loads but API fails | Backend issue |
| API works but UI fails | Frontend/config issue |

## Root Cause Thinking

500 happens because code crashed. Common production causes:

1. Missing environment variable

```text
DATABASE_URL missing
SECRET_KEY missing
JWT config missing
```

2. Database migration not applied

```text
column user_id does not exist
relation expenses does not exist
table users missing
```

3. Database connection failed

```text
wrong DB password
wrong DB host
DB container not running
production DB blocked
```

4. Dependency missing

Example FastAPI login issue:

```text
python-multipart not installed
```

5. New code expects data that old database does not have

Example:

```python
expense.user_id
```

but old rows do not have `user_id`.

6. Bad deployment build

```text
old backend image running
wrong branch deployed
wrong commit deployed
```

## Senior Debugging Approach

Do not guess. Follow this order.

### Step 1: Check if backend is alive

```bash
curl -i https://your-api-domain.com/health
```

Local/Docker:

```bash
curl -i http://127.0.0.1:8000/health
```

If health fails, backend/server itself has problem.

### Step 2: Check logs

Docker:

```bash
docker compose ps
docker compose logs api --tail=200
docker compose logs db --tail=100
```

Local uvicorn:

```bash
uvicorn app.main:app --reload
```

Then reproduce the issue and read the traceback in terminal.

You need the exact error line, not only “500”.

### Step 3: Check recent code change

```bash
git log --oneline -5
git show --stat HEAD
```

If issue started after deployment, recent change is highly suspicious.

### Step 4: Check migrations

```bash
alembic current
alembic heads
alembic upgrade head
```

If production DB schema is behind code, APIs can fail.

### Step 5: Reproduce with curl/Postman/Swagger

For login:

```bash
curl -X POST http://127.0.0.1:8000/auth/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=test@example.com&password=test123"
```

If curl fails, backend problem confirmed.

## Valid Solution

Once root cause is found, fix based on evidence.

### If migration missing

Run:

```bash
alembic upgrade head
```

Then restart backend.

### If env variable missing

Fix `.env` or production secrets:

```text
DATABASE_URL=...
SECRET_KEY=...
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Then redeploy/restart.

### If database connection failed

Check:

```bash
docker compose ps
docker compose logs db --tail=100
```

Verify DB host.

Inside Docker Compose, DB host should usually be:

```text
db
```

Not:

```text
localhost
127.0.0.1
```

### If new code caused crash

Rollback first if users are blocked:

```bash
git revert <bad_commit_hash>
```

Then redeploy stable version.

Fix properly in a new branch after service is stable.

## Prevention

Before deployment, always check:

```bash
pytest
npm test
npm run build
docker compose up --build
alembic upgrade head
```

Also maintain this deployment checklist:

```text
1. Correct branch?
2. Correct commit?
3. Env variables present?
4. DB reachable?
5. Migrations applied?
6. Health endpoint working?
7. Login working?
8. Main user flow tested?
```

## Practice Task

Take this issue:

```text
After deployment, POST /auth/login returns 500.
Swagger shows Internal Server Error.
```

Write the investigation in this format:

```text
Problem:
Impact:
Possible Root Causes:
First Command I Will Run:
Logs I Will Check:
Most Likely Fix:
Prevention:
```

Key rule for Day 1:

```text
500 error ki first medicine logs. Guess cheyyakudadhu.
```
