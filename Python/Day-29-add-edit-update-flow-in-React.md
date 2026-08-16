Regular `Day 29`: React full CRUD. Today you add `Edit / Update` to the React expense app.

**Day 29 Goal**

- Add `PUT` request from React
- Reuse form for add and edit
- Track selected expense in state
- Cancel editing
- Complete frontend CRUD: create, read, update, delete

Start from Day 28:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-28-react-router .\day-29-react-full-crud
cd day-29-react-full-crud
code .
```

Backend first:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-14-expense-api-review"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

In another terminal:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-29-react-full-crud"
npm install
npm run dev
```

Update `src/api/expenses.js` and add:

```javascript
export async function updateExpense(expenseId, expense) {
  const response = await fetch(`${API_URL}/expenses/${expenseId}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(expense),
  });

  if (!response.ok) {
    throw new Error("Failed to update expense.");
  }

  return response.json();
}
```

Replace `src/components/ExpenseForm.jsx`:

```jsx
import { useEffect, useState } from "react";

function ExpenseForm({ onSubmitExpense, editingExpense, onCancelEdit }) {
  const [form, setForm] = useState({
    title: "",
    amount: "",
    category: "",
  });

  useEffect(() => {
    if (editingExpense) {
      setForm({
        title: editingExpense.title,
        amount: editingExpense.amount,
        category: editingExpense.category,
      });
    }
  }, [editingExpense]);

  function handleChange(event) {
    const { name, value } = event.target;
    setForm({ ...form, [name]: value });
  }

  function handleSubmit(event) {
    event.preventDefault();

    onSubmitExpense({
      title: form.title.trim(),
      amount: Number(form.amount),
      category: form.category.trim(),
    });

    setForm({ title: "", amount: "", category: "" });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" value={form.title} onChange={handleChange} placeholder="Title" />
      <input name="amount" value={form.amount} onChange={handleChange} type="number" placeholder="Amount" />
      <input name="category" value={form.category} onChange={handleChange} placeholder="Category" />

      <button type="submit">
        {editingExpense ? "Update Expense" : "Add Expense"}
      </button>

      {editingExpense && (
        <button type="button" onClick={onCancelEdit}>
          Cancel
        </button>
      )}
    </form>
  );
}

export default ExpenseForm;
```

Update `src/components/ExpenseList.jsx`:

```jsx
function ExpenseList({ expenses, onDeleteExpense, onEditExpense }) {
  if (expenses.length === 0) {
    return <p>No expenses found.</p>;
  }

  return (
    <ul>
      {expenses.map((expense) => (
        <li key={expense.id}>
          <span>
            {expense.title} - ₹{expense.amount} - {expense.category}
          </span>

          <div>
            <button onClick={() => onEditExpense(expense)}>Edit</button>
            <button onClick={() => onDeleteExpense(expense.id)}>Delete</button>
          </div>
        </li>
      ))}
    </ul>
  );
}

export default ExpenseList;
```

In `src/pages/ExpensesPage.jsx`, import `updateExpense`:

```jsx
import {
  createExpense,
  getExpenses,
  removeExpense,
  updateExpense,
} from "../api/expenses";
```

Add state:

```jsx
const [editingExpense, setEditingExpense] = useState(null);
```

Replace `handleAddExpense` with:

```jsx
async function handleSubmitExpense(expense) {
  if (!expense.title || expense.amount <= 0 || !expense.category) {
    setMessage("Enter valid expense details.");
    return;
  }

  try {
    if (editingExpense) {
      await updateExpense(editingExpense.id, expense);
      setEditingExpense(null);
    } else {
      await createExpense(expense);
    }

    setMessage("");
    loadExpenses();
  } catch (error) {
    setMessage(error.message);
  }
}
```

Update component usage:

```jsx
<ExpenseForm
  onSubmitExpense={handleSubmitExpense}
  editingExpense={editingExpense}
  onCancelEdit={() => setEditingExpense(null)}
/>
```

Update `ExpenseList` usage:

```jsx
<ExpenseList
  expenses={expenses}
  onDeleteExpense={handleDeleteExpense}
  onEditExpense={setEditingExpense}
/>
```

Test:

- Add expense
- Click Edit
- Form should fill existing values
- Change amount/category
- Click Update Expense
- Refresh browser
- Updated data should remain
- Cancel edit should return form to add mode

Commit:

```powershell
git add .
git commit -m "Day 29 add edit update flow in React"
```

Day 29 is complete when you can explain: `editingExpense` tells React whether the form is creating a new record or updating an existing one.
