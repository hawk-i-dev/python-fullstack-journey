## App Skill Day 6 — Environment Config Mismatch

Technique:

```text
Feynman Technique + 80/20 Rule
```

Topic:

```text
Code works locally, but fails after deployment because environment/config values are different.
```

## 1. Feynman Meaning

Simple explanation:

```text
Your code is the same, but its settings changed.
If settings point to the wrong place, the app behaves differently.
```

Example:

Local frontend calls:

```text
http://localhost:8000
```

But production frontend should call:

```text
https://api.yourdomain.com
```

If production still calls `localhost`, user’s browser tries to call their own laptop, not your server.

## 2. Real-Time Problem

Common situation:

```text
Local app works.
Deployment successful.
Production app opens.
But login/API/database fails.
```

Usually because:

```text
API URL is wrong
DB host is wrong
JWT secret missing
CORS origin missing
Environment variable not loaded
Frontend was built with old env value
```

## 3. 80/20 Focus

Most config issues are found by checking these:

```text
1. Browser Network tab
2. Backend logs
3. Deployed env variables
4. API health endpoint
5. DB connection health
6. Frontend build-time config
```

Do these before changing code.

## 4. Common Symptoms

| Symptom | Likely Cause |
|---|---|
| Frontend calls `localhost` in production | Wrong frontend API URL |
| CORS error | Backend allowed origins missing frontend domain |
| Login gives `401` unexpectedly | JWT secret/token config mismatch |
| API gives `500` | Backend env/DB/config issue |
| DB connection failed | Wrong DB host/password/port |
| `API_URL undefined` | Env variable name missing/wrong |
| Works after local restart only | Env loaded at startup |

## 5. Important Concept: Frontend vs Backend Env

### Frontend env

For React/Vite:

```text
VITE_API_URL is used during build time.
```

Meaning:

```text
If you change VITE_API_URL after npm run build,
old value may still be inside built files.
```

Fix:

```bash
npm run build
```

Then redeploy frontend.

### Backend env

Backend usually reads env at runtime/startup.

If you change backend env:

```text
Restart/redeploy backend.
```

## 6. Safe Debugging Approach

### Step 1: Check browser Network tab

Open production app.

Inspect request URL.

Bad:

```text
http://localhost:8000/auth/login
```

Good:

```text
https://api.yourdomain.com/auth/login
```

If browser calls wrong URL, fix frontend config.

### Step 2: Check backend health

```bash
curl -i https://api.yourdomain.com/health
```

If DB health exists:

```bash
curl -i https://api.yourdomain.com/health/db
```

### Step 3: Check backend logs

Docker:

```bash
docker compose logs api --tail=200
```

Look for:

```text
DATABASE_URL missing
connection refused
authentication failed
invalid token
CORS blocked
```

### Step 4: Check env values safely

Do not expose secrets in chat or logs.

Check names/hosts, not full secrets:

```bash
docker compose exec api env | grep DB_HOST
docker compose exec api env | grep CORS
```

Avoid printing:

```text
SECRET_KEY
DB_PASSWORD
JWT_SECRET
```

## 7. Valid Solutions

### Case 1: Frontend calls localhost

Set correct production API URL.

Example:

```env
VITE_API_URL=https://api.yourdomain.com
```

Then rebuild:

```bash
npm run build
```

Redeploy frontend.

### Case 2: Backend cannot connect to DB

Local DB host:

```env
DB_HOST=localhost
```

Docker Compose DB host:

```env
DB_HOST=db
```

Production DB host:

```env
DB_HOST=your-production-db-host
```

Fix env, restart backend.

### Case 3: CORS error

Backend must allow frontend production URL.

Example:

```env
CORS_ORIGINS=https://yourdomain.com
```

Then restart backend.

### Case 4: JWT/token issue after deploy

Check same secret is used consistently:

```env
SECRET_KEY=production-secret
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

If secret changes, old tokens may become invalid. Users may need to login again.

## 8. What Not To Do

Avoid:

```text
Changing random frontend code
Hardcoding production URLs in code
Committing .env with secrets
Printing secrets in logs
Assuming local and prod are same
Redeploying without checking Network tab/logs
```

## 9. Prevention Checklist

Before deployment:

```text
1. .env.example updated
2. Required env vars documented
3. Dev/stage/prod configs separated
4. Frontend API URL verified
5. Backend DB host verified
6. CORS origins verified
7. Health endpoint tested
8. Login tested after deployment
```

## Practice Scenario

Issue:

```text
React app works locally.
After deployment, login fails.
Browser Network tab shows request going to:
http://localhost:8000/auth/login
```

Write:

```text
Problem:
Impact:
80/20 Focus:
Root Cause:
First Check:
Valid Solution:
Prevention:
```

Expected thinking:

```text
Production frontend was built with local API URL.
Set correct VITE_API_URL, rebuild frontend, redeploy, verify login.
```

Senior rule:

```text
Same code can fail with wrong config. Verify runtime settings.
```
