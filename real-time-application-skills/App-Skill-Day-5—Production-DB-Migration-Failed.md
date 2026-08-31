## App Skill Day 5 — Production DB Migration Failed

Technique:

```text
Feynman Technique + 80/20 Rule
```

Topic:

```text
Deployment happened, but database migration failed.
Now app code and database schema are not matching.
```

## 1. Feynman Meaning

Simple explanation:

```text
Application code expects the database to have a certain shape.
Migration changes that shape.
If migration fails, code may ask for a table or column that does not exist.
Then APIs can crash with 500 errors.
```

Example:

```python
user.full_name
```

But production DB still has only:

```text
users.name
```

Then app may fail because:

```text
full_name column does not exist
```

## 2. Real-Time Scenario

You deploy new backend code.

New code expects:

```text
expenses.user_id
categories.user_id
users.is_active
```

But migration did not run successfully.

Production error:

```text
column expenses.user_id does not exist
```

or:

```text
relation categories does not exist
```

or:

```text
duplicate key value violates unique constraint
```

This is a DB migration issue.

## 3. 80/20 Focus

Most migration failures are solved by focusing on these:

```text
1. Read exact DB error
2. Check current migration version
3. Compare app model vs DB schema
4. Protect production data first
5. Rollback app if users are blocked
6. Apply tested fix-forward migration
```

Do not randomly edit production database.

## 4. What You May See

Common errors:

```text
column does not exist
relation does not exist
duplicate key value violates unique constraint
cannot drop column because other objects depend on it
lock timeout
permission denied for schema
database is not accepting connections
```

Each error tells direction.

| Error | Meaning |
|---|---|
| `column does not exist` | Migration not applied or wrong DB |
| `relation does not exist` | Table missing |
| `duplicate key` | Existing data violates new unique rule |
| `cannot drop column` | Dependency exists |
| `lock timeout` | Migration blocked by active queries |
| `permission denied` | DB user lacks migration permission |

## 5. First Checks

### Check app logs

Docker:

```bash
docker compose logs api --tail=200
```

Server/system logs:

```bash
journalctl -u your-api-service -n 200
```

Cloud logs:

```text
Open deployment logs / runtime logs
```

Find the exact SQL error.

### Check migration status

For Alembic:

```bash
alembic current
alembic heads
alembic history --verbose
```

Meaning:

```text
current = migration applied in DB
heads = latest migration available in code
```

If current is behind heads, DB is behind code.

### Check DB schema

PostgreSQL:

```sql
\d expenses
\d categories
```

MySQL:

```sql
DESCRIBE expenses;
DESCRIBE categories;
```

Check whether expected columns exist.

## 6. Safe Decision: Rollback or Fix-Forward?

| Situation | Action |
|---|---|
| Users blocked and app code requires missing column | Rollback app first |
| Migration file is correct but not applied | Run migration |
| Migration has bad logic | Stop deployment, create fix-forward migration |
| Existing data violates new constraint | Clean/backfill data safely, then apply |
| Migration locks large table | Schedule safer migration window |
| Wrong DB was targeted | Stop immediately and verify config |

Senior rule:

```text
Stability first. Data safety always.
```

## 7. Valid Solutions

### Case 1: Migration not applied

If migration is correct and DB is simply behind:

```bash
alembic upgrade head
```

Then restart app if needed.

Verify:

```bash
alembic current
```

### Case 2: Bad app deployed before DB migration

Rollback app to previous stable version.

Then apply migration properly.

Then redeploy compatible app.

### Case 3: Existing data blocks migration

Example:

```text
You add NOT NULL column, but old rows have null.
```

Bad migration:

```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20) NOT NULL;
```

Better migration plan:

```text
1. Add nullable column
2. Backfill old rows
3. Update app to write new column
4. Later make column NOT NULL
```

This is called backward-compatible migration.

### Case 4: Unique constraint fails

Example:

```text
Adding unique constraint on email fails because duplicates already exist.
```

Approach:

```text
Find duplicates
Decide with business/team what to keep
Clean data safely
Then add constraint
```

Never delete production duplicates without approval.

## 8. Backward-Compatible Migration Pattern

For production, prefer this:

```text
Release 1:
- Add new nullable column
- App can work with old and new data

Release 2:
- Backfill data

Release 3:
- Make field required after all rows are valid
```

This avoids downtime.

## 9. Prevention Checklist

Before production migration:

```text
1. Backup exists
2. Migration tested on staging
3. Migration tested with production-like data size
4. Rollback plan ready
5. App version and DB version compatible
6. Large table locks considered
7. Migration reviewed
8. Smoke test after deploy
```

## 10. What Not To Do

Avoid:

```text
Editing prod DB manually without approval
Dropping columns quickly
Deleting production data to make migration pass
Running random SQL copied from internet
Ignoring lock/timeouts
Deploying app before schema is ready
```

## Practice Scenario

Issue:

```text
After deployment, GET /expenses returns 500.
Logs show: column expenses.user_id does not exist.
```

Write:

```text
Problem:
Impact:
80/20 Focus:
Most likely root cause:
First commands:
Rollback or fix-forward decision:
Valid solution:
Prevention:
```

Expected thinking:

```text
App code expects expenses.user_id.
Production DB does not have it.
Either migration was not applied or app deployed before migration.
Check alembic current vs heads.
If users blocked, rollback app or apply verified migration.
```

Final rule:

```text
Data safety first. Never edit production DB blindly.
```
