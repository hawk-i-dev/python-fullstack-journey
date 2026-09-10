Main issue: this screen looks like a dashboard template, not the Day 61 code we built. In Day 61, we only created frontend foundation. Expense adding comes in Day 63.

So if you are trying to add expense from this `+ Add Expense` button, it will not work unless that button is connected.

Do these checks in order.

## 1. First go to Expenses page

Click left side:

```text
Expenses
```

You should see an expense form with:

```text
Title
Amount
Category
Add expense button
Filters
Expense list
```

If you do not see that form, then Day 63 code is not added yet.

## 2. You must create category first

Expense requires `category_id`.

Before adding expense, create at least one category:

```text
Food
Travel
Bills
```

Either from frontend Categories page after Day 64, or from Swagger:

```text
http://127.0.0.1:8000/docs
```

Flow:

```text
Login
Authorize
POST /categories
POST /expenses
```

## 3. Fix dashboard `+ Add Expense` button

If you want dashboard button to open Expenses page, use this.

In your dashboard component:

```tsx
import { Link } from "react-router";
```

Then replace your button with:

```tsx
<Link to="/expenses" className="primary-button">
  + Add Expense
</Link>
```

Or if you are using a normal button:

```tsx
import { useNavigate } from "react-router";

const navigate = useNavigate();

<button onClick={() => navigate("/expenses")}>
  + Add Expense
</button>
```

## 4. Confirm route exists

In `src/App.tsx`, you must have:

```tsx
<Route path="/expenses" element={<ExpensesPage />} />
```

Inside protected routes:

```tsx
<Route element={<ProtectedRoute />}>
  <Route path="/dashboard" element={<DashboardPage />} />
  <Route path="/expenses" element={<ExpensesPage />} />
  <Route path="/categories" element={<CategoriesPage />} />
  <Route path="/reports" element={<ReportsPage />} />
  <Route path="/exports" element={<ExportsPage />} />
</Route>
```

## 5. If submit still fails, check browser Network tab

Press:

```text
F12 → Network → click Add Expense
```

Check `POST /expenses`.

Common errors:

```text
401 = token not sent / not logged in
404 = selected category does not belong to logged-in user
422 = request body wrong
500 = backend code error; check uvicorn terminal
```

Correct expense payload should look like:

```json
{
  "title": "Breakfast",
  "amount": "120.50",
  "category_id": 1
}
```

## 6. Backend and frontend must both run

Backend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Frontend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run dev
```

Bottom line: from this screenshot, the dashboard is showing, but expense adding is either not implemented yet or the `+ Add Expense` button is not connected. First complete Day 63 `ExpensesPage`, then connect this dashboard button to `/expenses`.
