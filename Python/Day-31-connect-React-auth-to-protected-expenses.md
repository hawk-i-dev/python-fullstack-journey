`Day 31`: connect React auth to protected expenses.

Today you combine Day 30 login with Day 21 protected expense API.

**Day 31 Goal**

- Use JWT token for expense API calls
- Load only logged-in user’s expenses
- Add expense after login
- Delete expense after login
- Load categories for dropdown
- Redirect unauthenticated users to login

Start backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

If React gets CORS error, add this to Day 21 `app/main.py`:

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

Now copy Day 30:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-30-react-auth .\day-31-react-protected-expenses
cd day-31-react-protected-expenses
code .
```

Create `src/api/expenses.js`:

```javascript
import { getToken } from "./auth";

const API_URL = import.meta.env.VITE_API_URL;

function authHeaders() {
  return {
    Authorization: `Bearer ${getToken()}`,
  };
}

export async function getCategories() {
  const response = await fetch(`${API_URL}/categories`);

  if (!response.ok) {
    throw new Error("Failed to load categories.");
  }

  return response.json();
}

export async function createCategory(category) {
  const response = await fetch(`${API_URL}/categories`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      ...authHeaders(),
    },
    body: JSON.stringify(category),
  });

  if (!response.ok) {
    throw new Error("Failed to create category.");
  }

  return response.json();
}

export async function getExpenses() {
  const response = await fetch(`${API_URL}/expenses`, {
    headers: authHeaders(),
  });

  if (!response.ok) {
    throw new Error("Failed to load expenses.");
  }

  return response.json();
}

export async function createExpense(expense) {
  const response = await fetch(`${API_URL}/expenses`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      ...authHeaders(),
    },
    body: JSON.stringify(expense),
  });

  if (!response.ok) {
    throw new Error("Failed to add expense.");
  }

  return response.json();
}

export async function deleteExpense(expenseId) {
  const response = await fetch(`${API_URL}/expenses/${expenseId}`, {
    method: "DELETE",
    headers: authHeaders(),
  });

  if (!response.ok) {
    throw new Error("Failed to delete expense.");
  }

  return response.json();
}
```

Create `src/pages/ExpensesPage.jsx`:

```jsx
import { useEffect, useState } from "react";
import { Link, useNavigate } from "react-router";
import {
  createExpense,
  deleteExpense,
  getCategories,
  getExpenses,
} from "../api/expenses";
import { getToken } from "../api/auth";

function ExpensesPage() {
  const navigate = useNavigate();
  const [expenses, setExpenses] = useState([]);
  const [categories, setCategories] = useState([]);
  const [form, setForm] = useState({ title: "", amount: "", category_id: "" });
  const [message, setMessage] = useState("");

  async function loadData() {
    try {
      setExpenses(await getExpenses());
      setCategories(await getCategories());
    } catch (error) {
      setMessage(error.message);
    }
  }

  useEffect(() => {
    if (!getToken()) {
      navigate("/login");
      return;
    }

    loadData();
  }, []);

  function handleChange(event) {
    setForm({ ...form, [event.target.name]: event.target.value });
  }

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

    await createExpense(expense);
    setForm({ title: "", amount: "", category_id: "" });
    setMessage("");
    loadData();
  }

  async function handleDelete(expenseId) {
    await deleteExpense(expenseId);
    loadData();
  }

  return (
    <>
      <h1>My Expenses</h1>

      <p>
        Need a category? Create it in Swagger first using <code>POST /categories</code>.
      </p>

      <form onSubmit={handleSubmit}>
        <input name="title" value={form.title} onChange={handleChange} placeholder="Title" />
        <input name="amount" value={form.amount} onChange={handleChange} type="number" placeholder="Amount" />

        <select name="category_id" value={form.category_id} onChange={handleChange}>
          <option value="">Select category</option>
          {categories.map((category) => (
            <option key={category.id} value={category.id}>
              {category.name}
            </option>
          ))}
        </select>

        <button type="submit">Add Expense</button>
      </form>

      {message && <p className="error">{message}</p>}

      <ul>
        {expenses.map((expense) => (
          <li key={expense.id}>
            <span>
              {expense.title} - ₹{expense.amount} - {expense.category.name}
            </span>
            <button onClick={() => handleDelete(expense.id)}>Delete</button>
          </li>
        ))}
      </ul>

      <Link to="/profile">Back to profile</Link>
    </>
  );
}

export default ExpensesPage;
```

Update `src/App.jsx`:

```jsx
import ExpensesPage from "./pages/ExpensesPage";
```

Add route:

```jsx
<Route path="/expenses" element={<ExpensesPage />} />
```

Add nav link:

```jsx
<Link to="/expenses">Expenses</Link>
```

Run React:

```powershell
npm install
npm run dev
```

Test order:

1. Register user
2. Login
3. Create category in Swagger if no category exists
4. Open `/expenses`
5. Add expense
6. Refresh page
7. Delete expense
8. Logout
9. Try `/expenses` again, should redirect to login

Commit:

```powershell
git add .
git commit -m "Day 31 connect React auth to protected expenses"
```

Day 31 is complete when you can explain: every protected expense request must send `Authorization: Bearer <token>`, and the backend uses that token to return only the logged-in user’s data.
