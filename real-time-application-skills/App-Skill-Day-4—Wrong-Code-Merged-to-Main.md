## App Skill Day 4 — Wrong Code Merged to Main

Technique used:

```text
Feynman Technique + 80/20 Rule
```

Topic:

```text
Bad code got merged into main branch.
Now CI, QA, or production is broken.
```

## 1. Feynman Meaning

Simple explanation:

```text
main branch is the shared road.
If bad code enters main, everyone using that road may face the problem.
```

So first goal is not “who made mistake?”

First goal is:

```text
Restore stable main.
```

## 2. Real-Time Scenario

Example:

```text
A PR was merged into main.
After merge, login started failing.
CI pipeline is red.
Deployment may already be triggered.
```

Symptoms:

```text
Tests failing
Build failing
API returning 500
Frontend broken
QA blocked
Production users affected
```

## 3. 80/20 Focus

Most cases are solved by these actions:

```text
1. Identify bad commit
2. Check impact
3. Stop deployment if needed
4. Revert safely or fix-forward
5. Run tests
6. Deploy verified code
7. Inform team
```

## 4. First Commands

Go to project:

```bash
git checkout main
git pull origin main
```

Check latest commits:

```bash
git log --oneline -10
```

Check what changed in suspicious commit:

```bash
git show <commit_hash>
```

Check current status:

```bash
git status
```

## 5. Decide: Revert or Fix-Forward?

| Situation | Best Action |
|---|---|
| Production users blocked | Revert first |
| Security/payment/data issue | Revert/disable feature immediately |
| CI broken on main | Revert or quick fix |
| Small typo/small safe bug | Fix-forward |
| Root cause unclear | Revert first, investigate later |

Senior decision:

```text
If impact is high, restore stability first.
```

## 6. Valid Solution: Safe Revert

Create a branch:

```bash
git checkout -b hotfix/revert-bad-main
```

If it was a normal commit:

```bash
git revert <commit_hash>
```

If it was a merge commit:

```bash
git revert -m 1 <merge_commit_hash>
```

Meaning:

```text
-m 1 means keep main branch as the base parent and undo the merged PR changes.
```

Then run tests:

```bash
pytest
npm test
npm run build
```

Push branch:

```bash
git push origin hotfix/revert-bad-main
```

Open PR:

```text
Title: Revert bad merge causing production issue
```

## 7. What Not To Do

Avoid:

```text
git reset --hard on shared main
force push main
delete random files
revert without checking impact
hide the issue from team
deploy without testing
```

Important:

```text
On shared branches, prefer git revert, not history rewrite.
```

## 8. Communication Template

Send this in team chat:

```text
Issue:
Latest merge to main appears to break login.

Impact:
CI failing and production deployment may be affected.

Action:
I am creating a revert PR to restore stable main.

Next:
After stability is restored, I will analyze root cause and raise a proper fix PR.
```

## 9. Prevention

Use this checklist:

```text
PR review required
CI must pass before merge
Small PRs
Feature flags for risky changes
No direct commits to main
Rollback plan ready
Smoke test after deployment
```

## Practice Task

Scenario:

```text
After PR merge, main branch build fails.
Production deployment is waiting.
Latest commit changed auth.py and database config.
```

Write:

```text
Problem:
Impact:
First command:
How to find bad commit:
Revert or fix-forward:
Commands:
Tests to run:
Team message:
Prevention:
```

Senior rule:

```text
Restore stability first. Then fix properly.
```
