You are on the Dashboard. From the screenshot, the `+ Add Expense` button is probably not connected yet.

In our Day 61–70 flow, expense creation is done from:

```text
Left sidebar → Expenses
```

Not directly from Dashboard unless you wire that button.

## First check

Click:

```text
Expenses
```

You should see the expense form:

```text
Title
Amount
Category
Add expense button
```

If you do not see that form, then `src/pages/ExpensesPage.tsx` was not replaced correctly from Day 63.

## Important condition

Before adding an expense, you must have at least one category.

Go to:

```text
Categories → Add category
```

Example:

```text
Food
Travel
Bills
```

Then go to:

```text
Expenses → Add expense
```

## Fix Dashboard `+ Add Expense` button

If you want the Dashboard button to work, update it to navigate to `/expenses`.

In `src/pages/DashboardPage.tsx`, use:

```tsx
import { Link } from "react-router";
```

Then replace the button with:

```tsx
<Link className="primary-button" to="/expenses">
  + Add Expense
</Link>
```

Or if it is already a button:

```tsx
import { useNavigate } from "react-router";

const navigate = useNavigate();

<button className="primary-button" onClick={() => navigate("/expenses")}>
  + Add Expense
</button>
```

## If expense form appears but save fails

Open browser DevTools → Network tab → click Add Expense → check status:

```text
401 = JWT token not sent
404 = selected category does not belong to logged-in user
422 = form data invalid
500 = backend error; check uvicorn terminal
```

Most common issue:

```text
category_id is empty or category was not created for current logged-in user
```

Correct expense payload should look like:

```json
{
  "title": "Breakfast",
  "amount": "120.50",
  "category_id": 1
}
```

So immediate next step:

```text
1. Login
2. Go to Categories
3. Add Food
4. Go to Expenses
5. Add expense using Food category
```

If you send me the error shown after clicking Add Expense, or the browser Network response, I can tell the exact fix.
