`Day 34`: search, filter, sort, and pagination in the protected React expense app.

Today you make the expense list usable when data grows.

**Day 34 Goal**

- Search expenses by title
- Filter by category
- Sort by amount/date
- Add pagination
- Keep JWT auth on every request
- Keep user-owned data protection

Start from Day 33:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-33-react-protected-edit .\day-34-react-filters-pagination
cd day-34-react-filters-pagination
code .
```

Backend: use the Day 22 `GET /expenses` logic if you already added it. If your Day 21 backend does not yet support query params, update `GET /expenses` in `app/routers/expenses.py` like this:

```python
@router.get("", response_model=list[ExpenseResponse])
def list_expenses(
    search: str | None = None,
    category_id: int | None = None,
    sort_by: str = "id",
    sort_order: str = "asc",
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    query = db.query(Expense).filter(Expense.user_id == current_user.id)

    if search:
        query = query.filter(Expense.title.ilike(f"%{search}%"))

    if category_id:
        query = query.filter(Expense.category_id == category_id)

    sort_columns = {
        "id": Expense.id,
        "amount": Expense.amount,
        "created_at": Expense.created_at,
    }

    sort_column = sort_columns.get(sort_by, Expense.id)

    if sort_order == "desc":
        query = query.order_by(sort_column.desc())
    else:
        query = query.order_by(sort_column.asc())

    return query.offset(skip).limit(limit).all()
```

Restart backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

In React, update `src/api/expenses.js`.

Replace `getExpenses()` with:

```javascript
export async function getExpenses(filters = {}) {
  const params = new URLSearchParams();

  if (filters.search) params.append("search", filters.search);
  if (filters.categoryId) params.append("category_id", filters.categoryId);
  if (filters.sortBy) params.append("sort_by", filters.sortBy);
  if (filters.sortOrder) params.append("sort_order", filters.sortOrder);
  params.append("skip", filters.skip ?? 0);
  params.append("limit", filters.limit ?? 10);

  const response = await fetch(`${API_URL}/expenses?${params.toString()}`, {
    headers: authHeaders(),
  });

  if (!response.ok) {
    throw new Error("Failed to load expenses.");
  }

  return response.json();
}
```

In `src/pages/ExpensesPage.jsx`, add filter state:

```jsx
const [filters, setFilters] = useState({
  search: "",
  categoryId: "",
  sortBy: "id",
  sortOrder: "asc",
  page: 1,
  limit: 5,
});
```

Update `loadData()`:

```jsx
async function loadData() {
  try {
    const skip = (filters.page - 1) * filters.limit;

    setExpenses(
      await getExpenses({
        search: filters.search,
        categoryId: filters.categoryId,
        sortBy: filters.sortBy,
        sortOrder: filters.sortOrder,
        skip,
        limit: filters.limit,
      })
    );

    setCategories(await getCategories());
  } catch (error) {
    setMessage(error.message);
  }
}
```

Update the effect:

```jsx
useEffect(() => {
  if (!getToken()) {
    navigate("/login");
    return;
  }

  loadData();
}, [filters]);
```

Add this above the expense form:

```jsx
<section className="filters">
  <input
    value={filters.search}
    onChange={(event) =>
      setFilters({ ...filters, search: event.target.value, page: 1 })
    }
    placeholder="Search by title"
  />

  <select
    value={filters.categoryId}
    onChange={(event) =>
      setFilters({ ...filters, categoryId: event.target.value, page: 1 })
    }
  >
    <option value="">All categories</option>
    {categories.map((category) => (
      <option key={category.id} value={category.id}>
        {category.name}
      </option>
    ))}
  </select>

  <select
    value={filters.sortBy}
    onChange={(event) =>
      setFilters({ ...filters, sortBy: event.target.value })
    }
  >
    <option value="id">Created order</option>
    <option value="amount">Amount</option>
    <option value="created_at">Date</option>
  </select>

  <select
    value={filters.sortOrder}
    onChange={(event) =>
      setFilters({ ...filters, sortOrder: event.target.value })
    }
  >
    <option value="asc">Ascending</option>
    <option value="desc">Descending</option>
  </select>
</section>
```

Add pagination below the expense list:

```jsx
<div className="pagination">
  <button
    disabled={filters.page === 1}
    onClick={() => setFilters({ ...filters, page: filters.page - 1 })}
  >
    Previous
  </button>

  <span>Page {filters.page}</span>

  <button
    disabled={expenses.length < filters.limit}
    onClick={() => setFilters({ ...filters, page: filters.page + 1 })}
  >
    Next
  </button>
</div>
```

Add CSS:

```css
.filters {
  display: grid;
  gap: 12px;
  margin-bottom: 20px;
}

.pagination {
  display: flex;
  gap: 12px;
  align-items: center;
  margin-top: 20px;
}
```

Run React:

```powershell
npm install
npm run dev
```

Test:

- Login
- Add at least 8 expenses
- Search by title
- Filter by category
- Sort by amount descending
- Use Next and Previous
- Confirm another user still cannot see your expenses

Commit:

```powershell
git add .
git commit -m "Day 34 add React filters sorting and pagination"
```

Day 34 is complete when you can explain: filtering must happen on the backend for real apps, because the frontend should not load unlimited records just to search locally.
