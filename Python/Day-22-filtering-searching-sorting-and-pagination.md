Day 22 is filtering, searching, sorting, and pagination. This is what real APIs need once data grows.

**Day 22 Goal**
You should understand:

- Query parameters
- Pagination with `skip` and `limit`
- Searching by text
- Filtering by category and amount
- Sorting API results
- Keeping user ownership protection in every query

Start by copying Day 21:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
Copy-Item -Recurse .\day-21-user-owned-expenses .\day-22-query-pagination
cd day-22-query-pagination
code .
```

Update `.env`:

```env
DB_NAME=python_fullstack_day22
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_HOST=localhost
DB_PORT=5432
JWT_SECRET_KEY=your_generated_secret_key
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Create database:

```sql
CREATE DATABASE python_fullstack_day22;
```

Run migrations:

```powershell
alembic upgrade head
```

Now update `GET /expenses` in `app/routers/expenses.py`.

```python
from fastapi import APIRouter, Depends, HTTPException, Query
```

Replace your current `list_expenses()` with:

```python
@router.get("", response_model=list[ExpenseResponse])
def list_expenses(
    search: str | None = None,
    category_id: int | None = None,
    min_amount: float | None = Query(default=None, ge=0),
    max_amount: float | None = Query(default=None, ge=0),
    sort_by: str = "id",
    sort_order: str = "asc",
    skip: int = Query(default=0, ge=0),
    limit: int = Query(default=10, ge=1, le=100),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    query = db.query(Expense).filter(Expense.user_id == current_user.id)

    if search:
        query = query.filter(Expense.title.ilike(f"%{search}%"))

    if category_id:
        query = query.filter(Expense.category_id == category_id)

    if min_amount is not None:
        query = query.filter(Expense.amount >= min_amount)

    if max_amount is not None:
        query = query.filter(Expense.amount <= max_amount)

    sort_columns = {
        "id": Expense.id,
        "title": Expense.title,
        "amount": Expense.amount,
        "created_at": Expense.created_at,
    }

    sort_column = sort_columns.get(sort_by)

    if sort_column is None:
        raise HTTPException(status_code=400, detail="Invalid sort field")

    if sort_order == "desc":
        query = query.order_by(sort_column.desc())
    elif sort_order == "asc":
        query = query.order_by(sort_column.asc())
    else:
        raise HTTPException(status_code=400, detail="Invalid sort order")

    return query.offset(skip).limit(limit).all()
```

Run:

```powershell
uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Test these after login/authorize:

```text
GET /expenses
GET /expenses?limit=5
GET /expenses?skip=5&limit=5
GET /expenses?search=tea
GET /expenses?category_id=1
GET /expenses?min_amount=100
GET /expenses?max_amount=500
GET /expenses?sort_by=amount&sort_order=desc
GET /expenses?search=tea&min_amount=10&sort_by=created_at&sort_order=desc
```

**Important Rule**
Every expense query must still include:

```python
Expense.user_id == current_user.id
```

Without that, pagination/search may leak another user’s data.

**Challenge**
Add a new endpoint:

```text
GET /expenses/count
```

It should return only the logged-in user’s expense count:

```json
{
  "count": 12
}
```

Hint:

```python
@router.get("/count")
def count_expenses(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    count = db.query(Expense).filter(Expense.user_id == current_user.id).count()
    return {"count": count}
```

Place `/count` above `/{expense_id}`.

Commit:

```powershell
git add .
git commit -m "Day 22 add filtering sorting and pagination"
```

Day 22 is complete when you can explain why APIs should not return unlimited records and why user filtering must happen before search, sort, and pagination.
