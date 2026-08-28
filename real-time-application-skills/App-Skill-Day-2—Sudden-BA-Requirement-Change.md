## App Skill Day 2 — Sudden BA Requirement Change

BA = Business Analyst.

Real-time scenario:

```text
You already started coding based on one requirement.
Suddenly BA says: “Small change only.”
But that “small change” may affect UI, API, DB, validation, tests, and deployment.
```

Senior developer rule:

```text
Clarify scope before changing code.
```

## Scenario Example

Old requirement:

```text
User must select category while creating expense.
```

New BA change:

```text
Category should be optional. If user does not select category, show it as Uncategorized.
```

This looks small, but it may affect:

```text
Frontend form
Backend validation
Database nullable column
Reports
Filters
Tests
Existing data
API response
```

## Problem

Sudden requirement change creates confusion because:

- current code may be half-built
- old acceptance criteria may be invalid
- tests may no longer match expected behavior
- developer may assume wrongly and create bugs

Do not immediately start editing code.

## Impact

Before coding, identify affected areas.

| Area | What to check |
|---|---|
| UI | Does form change? Field required or optional? |
| API | Request/response schema changes? |
| DB | Column nullable? New column? Migration needed? |
| Validation | Old validation still valid? |
| Tests | Existing test cases need update? |
| Reports | Any calculation affected? |
| Deployment | Safe to deploy now or needs feature flag? |

## Common Root Causes

Requirement changes usually happen because:

1. Business rule changed

```text
Client changed expected behavior.
```

2. BA missed edge case earlier

```text
What if category is not selected?
```

3. Stakeholder changed priority

```text
Release needs simpler flow first.
```

4. Production/user feedback came in

```text
Users are failing because category is mandatory.
```

5. Requirement was ambiguous

```text
“Category should be available” did not clearly say mandatory or optional.
```

## Correct Approach

### Step 1: Stop coding briefly

Do not continue based on old assumption.

First check:

```bash
git status
```

You need to know what files are already changed.

### Step 2: Ask what exactly changed

Ask BA this clearly:

```text
What was the old behavior?
What is the new behavior?
Is this change required for current release?
Which screens/API are affected?
Can you give one example input and expected output?
```

### Step 3: Convert change into acceptance criteria

Bad requirement:

```text
Make category optional.
```

Good acceptance criteria:

```text
1. User can create expense without category.
2. API should accept category_id as null.
3. Expense list should show "Uncategorized" when category is missing.
4. Reports should include uncategorized expenses.
5. Existing category-based expenses should continue working.
```

### Step 4: Do impact analysis

Example:

```text
Frontend:
- Remove required validation from category dropdown.
- Show "Uncategorized" in list.

Backend:
- category_id should be Optional.
- Do not reject missing category.

Database:
- category_id should allow NULL.
- Alembic migration may be needed.

Tests:
- Add test for expense without category.
- Keep old test for expense with category.
```

### Step 5: Confirm before changing code

Send this to BA/lead:

```text
Current understanding:
Category is now optional while creating expense.
If category is missing, we will save category_id as null and display "Uncategorized".
Reports will include those expenses under "Uncategorized".

Please confirm if this is correct.
```

This avoids rework.

## Valid Solution

Use a separate branch:

```bash
git checkout -b feature/category-optional
```

Before changing:

```bash
git status
git diff
```

Then change in small commits:

```text
Commit 1: backend schema/validation change
Commit 2: frontend form/list change
Commit 3: tests update
Commit 4: docs/ticket notes
```

Run tests:

```bash
pytest
npm test
npm run build
```

Then demo the changed flow:

```text
Create expense with category
Create expense without category
List expenses
Check report
Check edit flow
```

## What Not To Do

Avoid this:

```text
BA said change, so I changed random files quickly.
```

Also avoid:

```text
I assumed only frontend change.
```

Because requirement changes often affect backend and DB too.

## Prevention

For every requirement, ask for:

- acceptance criteria
- sample input/output
- edge cases
- validation rules
- permission rules
- report behavior
- release priority

Use this checklist before coding:

```text
1. Is the requirement clear?
2. Is there example data?
3. Are edge cases covered?
4. Is API behavior clear?
5. Is DB impact clear?
6. Are tests updated?
7. Is BA confirmation written in ticket/chat?
```

## Practice Task

Take this BA change:

```text
Old: Expense amount must be greater than 0.
New: Refund expenses can have negative amount.
```

Write your analysis:

```text
Problem:
Impact:
Questions to BA:
Affected files/modules:
Acceptance Criteria:
Valid Solution:
Tests Needed:
Prevention:
```

Key point:

```text
Requirement change small ani anipinchina, impact analysis lekunda code change cheyyakudadhu.
```
