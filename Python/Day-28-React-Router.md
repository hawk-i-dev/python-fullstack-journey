Day 28 is React Router. Today you add multiple pages to the React app instead of keeping everything on one screen.

Current React Router docs use `react-router` with Vite/React, so install that package. Source: [React Router installation](https://reactrouter.com/start/declarative/installation).

**Day 28 Goal**
You should understand:

- Client-side routing
- `BrowserRouter`
- `Routes`
- `Route`
- `NavLink`
- Page components
- Moving screen-level logic into `pages/`

Start by copying Day 27:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-27-react-components .\day-28-react-router
cd day-28-react-router
npm install react-router
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
├── pages/
│   ├── ExpensesPage.jsx
│   ├── ReportsPage.jsx
│   └── NotFoundPage.jsx
├── App.jsx
├── App.css
└── main.jsx
```

Replace `src/main.jsx`:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
);
```

Create `src/pages/ExpensesPage.jsx`:

```jsx
import { useEffect, useState } from "react";
import { createExpense, getExpenses, removeExpense } from "../api/expenses";
import ExpenseForm from "../components/ExpenseForm";
import ExpenseList from "../components/ExpenseList";
import Message from "../components/Message";

function ExpensesPage() {
  const [expenses, setExpenses] = useState([]);
  const [message, setMessage] = useState("");
  const [loading, setLoading] = useState(false);

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
    <>
      <h1>Expenses</h1>

      <ExpenseForm onAddExpense={handleAddExpense} />
      <Message text={message} />

      {loading ? (
        <p>Loading expenses...</p>
      ) : (
        <ExpenseList expenses={expenses} onDeleteExpense={handleDeleteExpense} />
      )}
    </>
  );
}

export default ExpensesPage;
```

Create `src/pages/ReportsPage.jsx`:

```jsx
import { useEffect, useState } from "react";
import { getExpenses } from "../api/expenses";
import Message from "../components/Message";

function ReportsPage() {
  const [expenses, setExpenses] = useState([]);
  const [message, setMessage] = useState("");

  useEffect(() => {
    async function loadReports() {
      try {
        setExpenses(await getExpenses());
      } catch (error) {
        setMessage(error.message);
      }
    }

    loadReports();
  }, []);

  const total = expenses.reduce((sum, expense) => sum + expense.amount, 0);

  return (
    <>
      <h1>Reports</h1>

      <Message text={message} />

      <h2>Total: ₹{total}</h2>
      <p>Expense count: {expenses.length}</p>
    </>
  );
}

export default ReportsPage;
```

Create `src/pages/NotFoundPage.jsx`:

```jsx
import { Link } from "react-router";

function NotFoundPage() {
  return (
    <>
      <h1>Page not found</h1>
      <p>The page you opened does not exist.</p>
      <Link to="/">Go to expenses</Link>
    </>
  );
}

export default NotFoundPage;
```

Replace `src/App.jsx`:

```jsx
import { NavLink, Route, Routes } from "react-router";
import ExpensesPage from "./pages/ExpensesPage";
import ReportsPage from "./pages/ReportsPage";
import NotFoundPage from "./pages/NotFoundPage";
import "./App.css";

function App() {
  return (
    <main className="container">
      <nav className="nav">
        <NavLink to="/">Expenses</NavLink>
        <NavLink to="/reports">Reports</NavLink>
      </nav>

      <Routes>
        <Route path="/" element={<ExpensesPage />} />
        <Route path="/reports" element={<ReportsPage />} />
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </main>
  );
}

export default App;
```

Add this to `src/App.css`:

```css
.nav {
  display: flex;
  gap: 16px;
  margin-bottom: 24px;
}

.nav a {
  color: #2563eb;
  text-decoration: none;
  font-weight: 600;
}

.nav a.active {
  text-decoration: underline;
}
```

Run backend first:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-14-expense-api-review"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run React in another terminal:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-28-react-router"
npm install
npm run dev
```

Test:

- `/` shows expenses
- `/reports` shows total and count
- Add an expense from `/`
- Go to `/reports`
- Total updates
- Open `/wrong-url`
- Not found page appears

Commit:

```powershell
git add .
git commit -m "Day 28 add React Router pages"
```

Day 28 is complete when you can explain this: React Router changes the visible page component without doing a full browser reload.
