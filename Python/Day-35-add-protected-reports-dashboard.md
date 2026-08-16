`Day 35`: protected reports dashboard.

Today you add backend report endpoints and show them in React. This is important because real dashboards should not calculate reports by loading all records into the frontend.

**Day 35 Goal**

- Add protected total/count endpoint
- Add protected category report endpoint
- Show dashboard in React
- Keep user ownership filtering
- Use backend aggregation with SQLAlchemy

Start from Day 34:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-34-react-filters-pagination .\day-35-react-reports-dashboard
cd day-35-react-reports-dashboard
code .
```

In Day 21 backend, update `app/routers/expenses.py`.

Add import:

```python
from sqlalchemy import func
```

Add these routes above `@router.get("/{expense_id}")`:

```python
@router.get("/summary")
def get_expense_summary(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    total = (
        db.query(func.coalesce(func.sum(Expense.amount), 0))
        .filter(Expense.user_id == current_user.id)
        .scalar()
    )

    count = db.query(Expense).filter(Expense.user_id == current_user.id).count()

    return {
        "total": float(total),
        "count": count,
    }


@router.get("/reports/by-category")
def get_expenses_by_category_report(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    rows = (
        db.query(Category.name, func.sum(Expense.amount))
        .join(Expense, Expense.category_id == Category.id)
        .filter(Expense.user_id == current_user.id)
        .group_by(Category.name)
        .order_by(func.sum(Expense.amount).desc())
        .all()
    )

    return [
        {
            "category": row[0],
            "total": float(row[1]),
        }
        for row in rows
    ]
```

Restart backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

In React, update `src/api/expenses.js`:

```javascript
export async function getExpenseSummary() {
  const response = await fetch(`${API_URL}/expenses/summary`, {
    headers: authHeaders(),
  });

  if (!response.ok) {
    throw new Error("Failed to load summary.");
  }

  return response.json();
}

export async function getCategoryReport() {
  const response = await fetch(`${API_URL}/expenses/reports/by-category`, {
    headers: authHeaders(),
  });

  if (!response.ok) {
    throw new Error("Failed to load category report.");
  }

  return response.json();
}
```

Create `src/pages/ReportsPage.jsx`:

```jsx
import { useEffect, useState } from "react";
import { getCategoryReport, getExpenseSummary } from "../api/expenses";

function ReportsPage() {
  const [summary, setSummary] = useState({ total: 0, count: 0 });
  const [categoryReport, setCategoryReport] = useState([]);
  const [message, setMessage] = useState("");

  useEffect(() => {
    async function loadReports() {
      try {
        setSummary(await getExpenseSummary());
        setCategoryReport(await getCategoryReport());
      } catch (error) {
        setMessage(error.message);
      }
    }

    loadReports();
  }, []);

  return (
    <>
      <h1>Reports</h1>

      {message && <p className="error">{message}</p>}

      <section className="summary-grid">
        <div className="summary-card">
          <h2>₹{summary.total}</h2>
          <p>Total expense</p>
        </div>

        <div className="summary-card">
          <h2>{summary.count}</h2>
          <p>Total records</p>
        </div>
      </section>

      <h2>By Category</h2>

      <ul>
        {categoryReport.map((item) => (
          <li key={item.category}>
            <span>{item.category}</span>
            <strong>₹{item.total}</strong>
          </li>
        ))}
      </ul>
    </>
  );
}

export default ReportsPage;
```

Update `src/App.jsx`:

```jsx
import ReportsPage from "./pages/ReportsPage";
```

Add nav link:

```jsx
<Link to="/reports">Reports</Link>
```

Add protected route:

```jsx
<Route path="/reports" element={<ReportsPage />} />
```

Add CSS:

```css
.summary-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin-bottom: 24px;
}

.summary-card {
  background: #f4f6f8;
  padding: 16px;
  border-radius: 8px;
}

.summary-card h2 {
  margin: 0;
}
```

Run React:

```powershell
npm install
npm run dev
```

Test:

- Login
- Add expenses in multiple categories
- Open `/reports`
- Total should match your expenses only
- Category totals should group correctly
- Login as another user and confirm reports are separate

Commit:

```powershell
git add .
git commit -m "Day 35 add protected reports dashboard"
```

Day 35 is complete when you can explain: reports should be calculated in the backend database query, and every report query must filter by `current_user.id`.
