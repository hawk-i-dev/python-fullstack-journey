## App Skill Day 7 — API Slow or Timing Out

Technique:

```text
Feynman Technique + 80/20 Rule
```

Topic:

```text
Production API suddenly becomes slow or starts timing out.
```

## 1. Feynman Meaning

Simple explanation:

```text
API is like a service counter.
If one request takes too long, users wait in line.
If many users keep waiting and retrying, the whole system feels down.
```

Example:

```text
Normal API response: 200 ms
Now same API response: 15 seconds
Some requests timeout with 504
```

That is a performance incident.

## 2. Real-Time Problem

Users report:

```text
Page keeps loading
Login is slow
Expense list takes too long
Reports never finish
API returns 504 Gateway Timeout
```

Important:

```text
Slow API is not always backend code only.
It can be DB, network, external API, huge data, or recent deployment.
```

## 3. 80/20 Focus

Most slow API incidents are found by checking:

```text
1. Which endpoint is slow?
2. Did it start after latest deployment?
3. Is database query slow?
4. Is response payload too large?
5. Is server CPU/memory high?
6. Is external API delaying response?
```

Do not optimize random code first.

## 4. What You See

Common symptoms:

| Symptom | Possible Cause |
|---|---|
| `504 Gateway Timeout` | Request took too long |
| Browser Network tab shows 20s+ | API/backend slow |
| DB CPU high | Slow query/missing index |
| App CPU high | Heavy processing loop |
| Memory high | Large response/data loading |
| Many duplicate requests | Frontend retry/useEffect issue |
| Only one endpoint slow | Specific query/code path issue |
| All endpoints slow | Server/DB/resource issue |

## 5. First Checks

### Check browser Network tab

Find:

```text
Which API is slow?
How long did it take?
Status code?
Payload size?
Is it called repeatedly?
```

### Check backend logs

Good logs should show duration:

```text
GET /expenses 200 180ms
GET /reports/monthly 504 30000ms
```

If logs don’t show duration, add request timing logs later.

### Check recent deployment

```bash
git log --oneline -5
git show --stat HEAD
```

Ask:

```text
Did this start after recent code change?
```

### Check database

If DB is suspected:

PostgreSQL:

```sql
EXPLAIN ANALYZE SELECT * FROM expenses WHERE user_id = 10;
```

MySQL:

```sql
EXPLAIN SELECT * FROM expenses WHERE user_id = 10;
```

Look for:

```text
full table scan
missing index
too many rows scanned
slow joins
```

## 6. Common Root Causes

### 1. Missing DB index

Example:

```sql
SELECT * FROM expenses WHERE user_id = 10;
```

If `user_id` has no index, DB may scan entire table.

Fix:

```sql
CREATE INDEX idx_expenses_user_id ON expenses(user_id);
```

### 2. N+1 query problem

Bad pattern:

```text
Get 100 expenses
Then query category 100 separate times
```

This creates 101 queries.

Fix:

```text
Use JOIN / eager loading
Fetch needed related data together
```

### 3. Large response payload

Bad:

```text
GET /expenses returns 50,000 rows
```

Fix:

```text
Pagination
Limit
Filters
Date range
```

### 4. External API delay

Example:

```text
Payment API / email API / third-party service is slow
```

Fix:

```text
Timeouts
Retries with limit
Queue background jobs
Fallback behavior
```

### 5. Frontend repeated calls

React issue:

```text
useEffect dependency wrong
API called repeatedly
```

Fix:

```text
Correct dependency array
Debounce search
Cancel old requests
```

## 7. Valid Solution Strategy

Choose based on proven bottleneck.

| Bottleneck | Solution |
|---|---|
| Missing index | Add index migration |
| Large data | Add pagination |
| N+1 queries | Use JOIN/eager loading |
| Heavy calculation | Cache or background job |
| External API slow | Timeout + async/queue |
| Too much traffic | Rate limit + scale |
| Bad recent deploy | Rollback or fix-forward |

Important:

```text
Scale only after checking root cause.
```

If DB query is bad, adding more servers may not fix it.

## 8. Safe Production Handling

If users are badly affected:

```text
1. Disable heavy feature temporarily
2. Rollback bad deploy if needed
3. Add temporary rate limit/cache
4. Fix root cause
5. Deploy tested fix
6. Monitor latency after deploy
```

## 9. Prevention

Add:

```text
Request duration logs
Slow query logs
Database indexes for filters
Pagination by default
Performance tests for large data
Alerts for latency and error rate
Dashboards for CPU/memory/DB
```

Deployment checklist:

```text
1. Did we add new list/report endpoint?
2. Does it paginate?
3. Does DB have indexes?
4. Does query avoid N+1?
5. Did we test with large data?
```

## Practice Scenario

Issue:

```text
After deployment, GET /expenses takes 25 seconds.
Network tab shows huge response.
Backend logs show 20,000 expenses returned.
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
The endpoint returns too much data.
Add pagination/filtering.
Do not load all rows at once.
Test with large dataset.
```

Senior rule:

```text
Measure first. Optimize the proven bottleneck.
```
