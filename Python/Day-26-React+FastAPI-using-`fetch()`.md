Day 26 is React + FastAPI using `fetch()`. Today your React UI will stop using local state only and start reading/writing real backend data.

Sources used for current guidance: React `useEffect` docs, Vite env variable docs, and MDN `fetch()` docs.  
React uses `useEffect` to sync with external systems like APIs; Vite exposes browser env variables only when they start with `VITE_`.

**Day 26 Goal**
You should understand:

- `useEffect`
- `fetch()`
- Loading state
- Error state
- `GET`, `POST`, `DELETE` from React
- React frontend talking to FastAPI backend
- `VITE_API_URL`

Start backend first. Use Day 14 API because it has no login:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-14-expense-api-review"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Keep that terminal open.

If CORS is not added in Day 14, add this in `app/main.py`:

```python
from fastapi.middleware.cors import CORSMiddleware
```

After `app = FastAPI(...)`:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173", "http://127.0.0.1:5173"],
    allow_credentials=False,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Now open a second PowerShell:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-25-react-basics .\day-26-react-fetch-api
cd day-26-react-fetch-api
code .
```

Create `.env` in the React project root:

```env
VITE_API_URL=http://127.0.0.1:8000
```

Replace `src/App.jsx`:

```jsx
import { useEffect, useState } from "react";
import "./App.css";

const API_URL = import.meta.env.VITE_API_URL;

function App() {
  const [expenses, setExpenses] = useState([]);
  const [form, setForm] = useState({
    title: "",
    amount: "",
    category: "",
  });
  const [message, setMessage] = useState("");
  const [loading, setLoading] = useState(false);

  const total = expenses.reduce((sum, expense) => sum + expense.amount, 0);

  async function loadExpenses() {
    try {
      setLoading(true);
      setMessage("");

      const response = await fetch(`${API_URL}/expenses`);

      if (!response.ok) {
        throw new Error("Failed to load expenses.");
      }

      const data = await response.json();
      setExpenses(data);
    } catch (error) {
      setMessage(error.message);
    } finally {
      setLoading(false);
    }
  }

  useEffect(() => {
    loadExpenses();
  }, []);

  function handleChange(event) {
    const { name, value } = event.target;

    setForm({
      ...form,
      [name]: value,
    });
  }

  async function handleSubmit(event) {
    event.preventDefault();

    const newExpense = {
      title: form.title.trim(),
      amount: Number(form.amount),
      category: form.category.trim(),
    };

    if (!newExpense.title || newExpense.amount <= 0 || !newExpense.category) {
      setMessage("Enter valid expense details.");
      return;
    }

    try {
      const response = await fetch(`${API_URL}/expenses`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(newExpense),
      });

      if (!response.ok) {
        throw new Error("Failed to add expense.");
      }

      setForm({ title: "", amount: "", category: "" });
      setMessage("");
      loadExpenses();
    } catch (error) {
      setMessage(error.message);
    }
  }

  async function deleteExpense(expenseId) {
    try {
      const response = await fetch(`${API_URL}/expenses/${expenseId}`, {
        method: "DELETE",
      });

      if (!response.ok) {
        throw new Error("Failed to delete expense.");
      }

      loadExpenses();
    } catch (error) {
      setMessage(error.message);
    }
  }

  return (
    <main className="container">
      <h1>React + FastAPI Expense Tracker</h1>

      <form onSubmit={handleSubmit}>
        <input
          name="title"
          value={form.title}
          onChange={handleChange}
          placeholder="Title"
        />

        <input
          name="amount"
          value={form.amount}
          onChange={handleChange}
          type="number"
          placeholder="Amount"
        />

        <input
          name="category"
          value={form.category}
          onChange={handleChange}
          placeholder="Category"
        />

        <button type="submit">Add Expense</button>
      </form>

      {message && <p className="error">{message}</p>}

      <h2>Total: ₹{total}</h2>

      {loading ? (
        <p>Loading expenses...</p>
      ) : (
        <ul>
          {expenses.map((expense) => (
            <li key={expense.id}>
              <span>
                {expense.title} - ₹{expense.amount} - {expense.category}
              </span>

              <button onClick={() => deleteExpense(expense.id)}>Delete</button>
            </li>
          ))}
        </ul>
      )}
    </main>
  );
}

export default App;
```

Run React:

```powershell
npm install
npm run dev
```

Open:

```text
http://localhost:5173
```

Test:

- Add `Tea`, `20`, `Food`
- Add `Bus`, `35`, `Travel`
- Refresh browser
- Data should still show
- Delete one expense
- Check FastAPI Swagger also reflects the same data

**Key Concept**
This runs once when the React component loads:

```jsx
useEffect(() => {
  loadExpenses();
}, []);
```

The empty `[]` means: run after first render only.

Commit:

```powershell
git add .
git commit -m "Day 26 connect React frontend to FastAPI API"
```

Day 26 is complete when you can explain this flow:

`React component loads -> useEffect calls fetch -> FastAPI reads PostgreSQL -> React stores response in state -> UI re-renders.`
