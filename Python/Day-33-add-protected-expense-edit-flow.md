`Day 33`: edit/update protected expenses.

Today you complete authenticated expense CRUD. Until now React can create/list/delete expenses. Now it can edit an existing expense using `PUT /expenses/{expense_id}`.

**Day 33 Goal**

- Add protected backend `PUT /expenses/{expense_id}` if missing
- Add `updateExpense()` API function in React
- Fill form when clicking Edit
- Update title, amount, category
- Cancel edit
- Keep user ownership protection

Start from Day 32:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-32-react-categories .\day-33-react-protected-edit
cd day-33-react-protected-edit
code .
```

First make sure Day 21 backend has `PUT /expenses/{expense_id}`.

In Day 21 backend `app/schemas.py`, add if missing:

```python
class ExpenseUpdate(BaseModel):
    title: str = Field(min_length=1)
    amount: float = Field(gt=0)
    category_id: int
```

In `app/routers/expenses.py`, add/update this route:

```python
@router.put("/{expense_id}", response_model=ExpenseResponse)
def update_expense(
    expense_id: int,
    expense_data: ExpenseUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    category = db.query(Category).filter(Category.id == expense_data.category_id).first()

    if category is None:
        raise HTTPException(status_code=404, detail="Category not found")

    expense = (
        db.query(Expense)
        .filter(Expense.id == expense_id, Expense.user_id == current_user.id)
        .first()
    )

    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")

    expense.title = expense_data.title.strip()
    expense.amount = expense_data.amount
    expense.category_id = expense_data.category_id

    db.commit()
    db.refresh(expense)

    return expense
```

Also import `ExpenseUpdate`:

```python
from app.schemas import ExpenseCreate, ExpenseResponse, ExpenseUpdate
```

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Now React changes.

In `src/api/expenses.js`, add:

```javascript
export async function updateExpense(expenseId, expense) {
  const response = await fetch(`${API_URL}/expenses/${expenseId}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
      ...authHeaders(),
    },
    body: JSON.stringify(expense),
  });

  if (!response.ok) {
    throw new Error("Failed to update expense.");
  }

  return response.json();
}
```

In `src/pages/ExpensesPage.jsx`, import it:

```jsx
import {
  createExpense,
  deleteExpense,
  getCategories,
  getExpenses,
  updateExpense,
} from "../api/expenses";
```

Add state:

```jsx
const [editingExpense, setEditingExpense] = useState(null);
```

Update `handleSubmit` so it creates or updates:

```jsx
async function handleSubmit(event) {
  event.preventDefault();

  const expense = {
    title: form.title.trim(),
    amount: Number(form.amount),
    category_id: Number(form.category_id),
  };

  if (!expense.title || expense.amount <= 0 || !expense.category_id) {
    setMessage("Enter valid expense details.");
    return;
  }

  if (editingExpense) {
    await updateExpense(editingExpense.id, expense);
    setEditingExpense(null);
  } else {
    await createExpense(expense);
  }

  setForm({ title: "", amount: "", category_id: "" });
  setMessage("");
  loadData();
}
```

Add edit handler:

```jsx
function handleEdit(expense) {
  setEditingExpense(expense);

  setForm({
    title: expense.title,
    amount: expense.amount,
    category_id: expense.category.id,
  });
}
```

Change submit button:

```jsx
<button type="submit">
  {editingExpense ? "Update Expense" : "Add Expense"}
</button>
```

Add cancel button below it:

```jsx
{editingExpense && (
  <button
    type="button"
    onClick={() => {
      setEditingExpense(null);
      setForm({ title: "", amount: "", category_id: "" });
    }}
  >
    Cancel
  </button>
)}
```

In the expenses list, add Edit button:

```jsx
<button onClick={() => handleEdit(expense)}>Edit</button>
<button onClick={() => handleDelete(expense.id)}>Delete</button>
```

Run React:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-33-react-protected-edit"
npm install
npm run dev
```

Test:

- Login
- Create categories
- Add expense
- Click Edit
- Form should fill old data
- Change amount/category
- Click Update Expense
- Refresh page
- Updated data should remain
- Login as another user and confirm they cannot edit your expense

Commit:

```powershell
git add .
git commit -m "Day 33 add protected expense edit flow"
```

Day 33 is complete when you can explain: update queries must filter by both `expense_id` and `current_user.id`, otherwise one user could update another user’s expense.
