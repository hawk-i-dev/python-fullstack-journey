## App Skill Day 8 — Frontend Blank Screen After Deployment

Technique:

```text
Feynman Technique + 80/20 Rule
```

Topic:

```text
React/frontend app works locally, but production shows a blank white screen after deployment.
```

## 1. Feynman Meaning

Simple explanation:

```text
The HTML page loaded, but JavaScript crashed before drawing the UI.
```

Think like this:

```text
Shop is open, door is unlocked, but lights are off.
Users can enter the URL, but they cannot see/use the app.
```

That is a frontend blank screen issue.

## 2. Real-Time Problem

Users report:

```text
Website opens but shows blank page
Dashboard is white screen
Nothing happens after login
Page refresh on /dashboard gives 404
Works locally but fails in production
```

This is usually not solved by staring at code first.

You inspect browser evidence.

## 3. 80/20 Focus

Most blank screen issues are found using:

```text
1. Browser Console
2. Browser Network tab
3. Latest deployment/commit
4. Frontend env variables
5. Build output/static asset paths
```

Order matters:

```text
Console first, Network second.
```

## 4. What You See

Common symptoms:

| Symptom | Likely Cause |
|---|---|
| Red JS error in Console | React/runtime crash |
| `Cannot read property of undefined` | Data shape mismatch |
| JS/CSS file 404 | Static asset path/deploy issue |
| `/dashboard` refresh gives 404 | SPA routing not configured |
| API URL is `undefined` | Env variable missing |
| API calls `localhost` | Wrong production API URL |
| Blank only after login | Auth/user data issue |
| Blank only one page | Component-specific bug |

## 5. First Checks

### Step 1: Open browser Console

In Chrome/Edge:

```text
Right click → Inspect → Console
```

Look for red errors like:

```text
Uncaught TypeError
Cannot read properties of undefined
ReferenceError
Failed to load module script
```

This tells which file/component crashed.

### Step 2: Open Network tab

Check:

```text
Are JS/CSS files loading?
Any 404 or 500?
Which API call failed?
Is frontend calling localhost?
Is payload different from expected?
```

### Step 3: Check latest deployment

```bash
git log --oneline -5
git show --stat HEAD
```

Ask:

```text
Did blank screen start after this commit?
```

## 6. Common Root Causes

### 1. Runtime JavaScript error

Example:

```javascript
const name = user.profile.name;
```

But production response:

```json
{
  "user": null
}
```

Then app crashes.

Safer code:

```javascript
const name = user?.profile?.name || "User";
```

### 2. API response shape changed

Frontend expects:

```json
{
  "items": []
}
```

Backend sends:

```json
[]
```

Frontend crashes because it tries:

```javascript
data.items.map(...)
```

Fix:

```javascript
const expenses = Array.isArray(data) ? data : data.items;
```

Better: align frontend and backend contract.

### 3. Wrong production env

Bad:

```text
VITE_API_URL=http://localhost:8000
```

Good:

```text
VITE_API_URL=https://api.yourdomain.com
```

After changing Vite env:

```bash
npm run build
```

Then redeploy.

### 4. Static assets not found

Network tab shows:

```text
/main.js 404
/style.css 404
```

Causes:

```text
Wrong base path
Wrong hosting config
Build folder not uploaded correctly
```

### 5. SPA route refresh 404

React route works when clicking, but refresh fails:

```text
/dashboard → 404
```

Reason:

```text
Server does not know React routes.
```

Fix:

```text
Configure SPA fallback to serve index.html.
```

For Nginx:

```nginx
try_files $uri $uri/ /index.html;
```

## 7. Valid Solution

Use evidence-based fix.

| Cause | Valid Solution |
|---|---|
| JS runtime error | Fix exact component/data handling |
| API shape mismatch | Align API contract or adapt parser |
| Wrong env | Set correct env, rebuild, redeploy |
| Static asset 404 | Fix build/deploy/static path |
| Route refresh 404 | Configure SPA fallback |
| Bad latest deploy | Rollback or fix-forward |

After fix:

```bash
npm run build
npm test
```

Then smoke test:

```text
Open homepage
Login
Open dashboard
Refresh dashboard
Call main API flow
Logout/login again
```

## 8. What Not To Do

Avoid:

```text
Changing random React files
Ignoring Console error
Only clearing cache and assuming fixed
Hardcoding production URL inside code
Deploying without npm run build
Skipping route refresh test
```

## 9. Prevention

Use this checklist:

```text
1. npm run build passes
2. Frontend env verified
3. Console has no red error
4. Network tab has no missing JS/CSS
5. SPA fallback configured
6. Error boundary added
7. E2E smoke test after deployment
8. Frontend error monitoring enabled
```

## Practice Scenario

Issue:

```text
After deployment, React app shows blank screen.
Console shows:
Cannot read properties of undefined reading 'map'
```

Write:

```text
Problem:
Impact:
80/20 Focus:
Likely Root Cause:
First Checks:
Valid Solution:
Prevention:
```

Expected thinking:

```text
Frontend tried to map undefined data.
Check API response shape.
Add safe default like [].
Align backend response with frontend expectation.
Add test for empty/loading/error states.
```

Senior rule:

```text
Blank screen? Console first, Network second.
```
