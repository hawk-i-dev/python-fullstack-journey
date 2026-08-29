## App Skill Day 3 — Git Conflict During Urgent Fix

Today we apply:

```text
Feynman Technique + 80/20 Rule
```

Topic:

```text
Git conflict during urgent production fix
```

## 1. Feynman Meaning

Simple explanation:

```text
Two people edited the same part of the same file.
Git cannot decide whose change is correct.
So Git stops and asks a human to choose the final version.
```

Think of it like this:

```text
Two people edited the same sentence in a document.
One wrote version A.
Another wrote version B.
Now someone must create the final correct sentence.
```

That is a Git conflict.

## 2. Real-Time Scenario

You are working on a bug fix:

```text
Bug: Login fails after deployment.
Branch: hotfix/login-500
File: auth.py
```

You try to pull latest code:

```bash
git pull origin main
```

or rebase:

```bash
git rebase main
```

Git says:

```text
CONFLICT (content): Merge conflict in app/routers/auth.py
Automatic merge failed; fix conflicts and then commit the result.
```

This is common in real projects.

## 3. What You See in File

Git marks conflict like this:

```text
<<<<<<< HEAD
your current code
=======
incoming code from main
>>>>>>> main
```

Meaning:

| Marker | Meaning |
|---|---|
| `<<<<<<< HEAD` | Your current branch code starts |
| `=======` | Separator between two versions |
| `>>>>>>> main` | Incoming branch code ends |

Do not leave these markers in final code.

## 4. 80/20 Focus

For most Git conflicts, these few actions solve the issue:

```text
1. Run git status
2. Open conflicted files
3. Understand both versions
4. Keep correct final code
5. Remove conflict markers
6. Run tests
7. git add resolved files
8. Continue merge/rebase
```

This solves most day-to-day conflicts.

## 5. Safe Approach

### Step 1: Check status

```bash
git status
```

You will see files like:

```text
both modified: app/routers/auth.py
```

These files need manual resolution.

### Step 2: Open file in VS Code

```bash
code app/routers/auth.py
```

VS Code may show buttons:

```text
Accept Current Change
Accept Incoming Change
Accept Both Changes
Compare Changes
```

Do not blindly click.

Understand both sides first.

## 6. How to Decide Correct Code

Ask:

```text
What is my hotfix trying to fix?
What did main branch change?
Can both changes safely exist together?
Will this break existing behavior?
```

Example conflict:

```python
<<<<<<< HEAD
def login(username: str, password: str):
    user = get_user_by_username(username)
=======
def login(email: str, password: str):
    user = get_user_by_email(email)
>>>>>>> main
```

Bad resolution:

```python
def login(username: str, password: str):
    user = get_user_by_username(username)
```

Maybe main changed login from username to email. If you ignore that, you break latest requirement.

Better resolution may be:

```python
def login(email: str, password: str):
    user = get_user_by_email(email)
```

Then apply your hotfix logic inside the latest structure.

## 7. Valid Solution Flow

If you are doing merge:

```bash
git status
# fix files manually
git add app/routers/auth.py
git commit
```

If you are doing rebase:

```bash
git status
# fix files manually
git add app/routers/auth.py
git rebase --continue
```

If rebase becomes too confusing and you have not pushed bad changes, ask lead before aborting:

```bash
git rebase --abort
```

Do not run destructive commands randomly.

## 8. After Resolving Conflict

Run relevant tests:

Backend:

```bash
pytest
```

Frontend:

```bash
npm test
npm run build
```

For Docker app:

```bash
docker compose up --build
```

Also test the actual bug:

```text
Login works
Old login behavior still works
Health endpoint works
No new 500 error
```

## 9. What Not To Do

Avoid:

```text
Accept Current Change blindly
Accept Incoming Change blindly
Remove random lines
Commit conflict markers
Skip tests because fix is urgent
Force push without understanding
```

Urgent fix does not mean unsafe fix.

## 10. Prevention

To reduce conflicts:

```text
Pull latest before starting work
Keep branches short-lived
Commit small logical changes
Avoid many people editing same file silently
Communicate when touching risky files
Review diff before merge
```

Useful commands:

```bash
git diff
git diff --staged
git log --oneline -5
```

## 11. Practice Scenario

You are on:

```text
hotfix/login-500
```

Conflict file:

```text
app/routers/auth.py
```

Conflict:

```python
<<<<<<< HEAD
raise HTTPException(status_code=500, detail="Login failed")
=======
raise HTTPException(status_code=401, detail="Invalid credentials")
>>>>>>> main
```

Question:

```text
Which final code should you keep and why?
```

Expected answer:

```python
raise HTTPException(status_code=401, detail="Invalid credentials")
```

Reason:

```text
Invalid login is not server crash.
It is unauthorized user input.
Correct status code is 401, not 500.
```

## Final Senior Rule

```text
Protect work first. Resolve conflict with understanding, not button-clicking.
```
