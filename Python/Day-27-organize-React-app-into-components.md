**Day 27 Goal**
Clean up the Day 26 React app by splitting it into:

- API functions
- Components
- Main app state

Start by copying Day 26:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-26-react-fetch-api .\day-27-react-components
cd day-27-react-components
code .
```

Create this structure:

```text
src/
├── api/
│   └── expenses.js
├── components/
│   ├── ExpenseForm.jsx
│   ├── ExpenseList.jsx
│   └── Message.jsx
├── App.jsx
├── App.css
└── main.jsx
```

`src/api/expenses.js`:

```javascript
const API_URL = import.meta.env.VITE_API_URL;

export async function getExpenses() {
  const response = await fetch(`${API_URL}/expenses`);

  if (!response.ok) {
    throw new Error("Failed to load expenses.");
  }

  return response.json();
}

export async function createExpense(expense) {
  const response = await fetch(`${API_URL}/expenses`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(expense),
  });

  if (!response.ok) {
    throw new Error("Failed to add expense.");
  }

  return response.json();
}

export async function removeExpense(expenseId) {
  const response = await fetch(`${API_URL}/expenses/${expenseId}`, {
    method: "DELETE",
  });

  if (!response.ok) {
    throw new Error("Failed to delete expense.");
  }

  return response.json();
}
```

`src/components/ExpenseForm.jsx`:

```jsx
import { useState } from "react";

function ExpenseForm({ onAddExpense }) {
  const [form, setForm] = useState({
    title: "",
    amount: "",
    category: "",
  });

  function handleChange(event) {
    const { name, value } = event.target;
    setForm({ ...form, [name]: value });
  }

  function handleSubmit(event) {
    event.preventDefault();

    onAddExpense({
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
      <button type="submit">Add Expense</button>
    </form>
  );
}

export default ExpenseForm;
```

`src/components/ExpenseList.jsx`:

```jsx
function ExpenseList({ expenses, onDeleteExpense }) {
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
          <button onClick={() => onDeleteExpense(expense.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

export default ExpenseList;
```

`src/components/Message.jsx`:

```jsx
function Message({ text }) {
  if (!text) {
    return null;
  }

  return <p className="error">{text}</p>;
}

export default Message;
```

Replace `src/App.jsx`:

```jsx
import { useEffect, useState } from "react";
import { createExpense, getExpenses, removeExpense } from "./api/expenses";
import ExpenseForm from "./components/ExpenseForm";
import ExpenseList from "./components/ExpenseList";
import Message from "./components/Message";
import "./App.css";

function App() {
  const [expenses, setExpenses] = useState([]);
  const [message, setMessage] = useState("");
  const [loading, setLoading] = useState(false);

  const total = expenses.reduce((sum, expense) => sum + expense.amount, 0);

  async function loadExpenses() {
    try {
      setLoading(true);
      setMessage("");
      setExpenses(await getExpenses());
    } catch (error) {
      setMessage(error.message);
    } finally {
      setLoading(false);
    }
  }

  useEffect(() => {
    loadExpenses();
  }, []);

  async function handleAddExpense(expense) {
    if (!expense.title || expense.amount <= 0 || !expense.category) {
      setMessage("Enter valid expense details.");
      return;
    }

    await createExpense(expense);
    loadExpenses();
  }

  async function handleDeleteExpense(expenseId) {
    await removeExpense(expenseId);
    loadExpenses();
  }

  return (
    <main className="container">
      <h1>React Expense Tracker</h1>

      <ExpenseForm onAddExpense={handleAddExpense} />
      <Message text={message} />

      <h2>Total: ₹{total}</h2>

      {loading ? (
        <p>Loading expenses...</p>
      ) : (
        <ExpenseList expenses={expenses} onDeleteExpense={handleDeleteExpense} />
      )}
    </main>
  );
}

export default App;
```

Run backend first:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-14-expense-api-review"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run React in another terminal:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-27-react-components"
npm install
npm run dev
```

Commit:

```powershell
git add .
git commit -m "Day 27 organize React app into components"
```

Day 27 is complete when you can explain why API calls belong in `src/api`, while UI belongs in `src/components`.
