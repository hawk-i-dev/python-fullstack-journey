`Day 37`: React cleanup with custom hooks and reusable layout.

Today you reduce repeated logic in the React app. This is a professional frontend skill: pages should not become huge because every page manages API loading, errors, and auth manually.

**Day 37 Goal**

- Create reusable layout
- Create `useAuth()` hook
- Create `useReports()` hook
- Keep pages smaller
- Centralize logout/auth behavior
- Make the app easier to maintain

Start from Day 36:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-36-react-report-charts .\day-37-react-custom-hooks
cd day-37-react-custom-hooks
code .
```

Create folders:

```text
src/
├── components/
│   ├── AppLayout.jsx
│   └── ProtectedRoute.jsx
├── hooks/
│   ├── useAuth.js
│   └── useReports.js
```

Create `src/hooks/useAuth.js`:

```javascript
import { getToken, logout } from "../api/auth";

export function useAuth() {
  const isLoggedIn = Boolean(getToken());

  function logoutUser() {
    logout();
  }

  return {
    isLoggedIn,
    logoutUser,
  };
}
```

Create `src/hooks/useReports.js`:

```javascript
import { useEffect, useState } from "react";
import { getCategoryReport, getExpenseSummary } from "../api/expenses";

export function useReports() {
  const [summary, setSummary] = useState({ total: 0, count: 0 });
  const [categoryReport, setCategoryReport] = useState([]);
  const [message, setMessage] = useState("");
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    async function loadReports() {
      try {
        setLoading(true);
        setMessage("");
        setSummary(await getExpenseSummary());
        setCategoryReport(await getCategoryReport());
      } catch (error) {
        setMessage(error.message);
      } finally {
        setLoading(false);
      }
    }

    loadReports();
  }, []);

  return {
    summary,
    categoryReport,
    message,
    loading,
  };
}
```

Create `src/components/AppLayout.jsx`:

```jsx
import { Link, useNavigate } from "react-router";
import { useAuth } from "../hooks/useAuth";

function AppLayout({ children }) {
  const navigate = useNavigate();
  const { isLoggedIn, logoutUser } = useAuth();

  function handleLogout() {
    logoutUser();
    navigate("/login");
  }

  return (
    <main className="container">
      <nav className="nav">
        <Link to="/register">Register</Link>
        <Link to="/login">Login</Link>

        {isLoggedIn && (
          <>
            <Link to="/profile">Profile</Link>
            <Link to="/categories">Categories</Link>
            <Link to="/expenses">Expenses</Link>
            <Link to="/reports">Reports</Link>
            <button onClick={handleLogout}>Logout</button>
          </>
        )}
      </nav>

      {children}
    </main>
  );
}

export default AppLayout;
```

Update `src/App.jsx`:

```jsx
import { Route, Routes } from "react-router";
import AppLayout from "./components/AppLayout";
import ProtectedRoute from "./components/ProtectedRoute";
import CategoriesPage from "./pages/CategoriesPage";
import ExpensesPage from "./pages/ExpensesPage";
import LoginPage from "./pages/LoginPage";
import ProfilePage from "./pages/ProfilePage";
import RegisterPage from "./pages/RegisterPage";
import ReportsPage from "./pages/ReportsPage";
import "./App.css";

function App() {
  return (
    <AppLayout>
      <Routes>
        <Route path="/" element={<LoginPage />} />
        <Route path="/register" element={<RegisterPage />} />
        <Route path="/login" element={<LoginPage />} />

        <Route element={<ProtectedRoute />}>
          <Route path="/profile" element={<ProfilePage />} />
          <Route path="/categories" element={<CategoriesPage />} />
          <Route path="/expenses" element={<ExpensesPage />} />
          <Route path="/reports" element={<ReportsPage />} />
        </Route>
      </Routes>
    </AppLayout>
  );
}

export default App;
```

Update `src/pages/ReportsPage.jsx` to use the hook:

```jsx
import {
  Bar,
  BarChart,
  CartesianGrid,
  Cell,
  Legend,
  Pie,
  PieChart,
  ResponsiveContainer,
  Tooltip,
  XAxis,
  YAxis,
} from "recharts";
import { useReports } from "../hooks/useReports";

const COLORS = ["#2563eb", "#16a34a", "#dc2626", "#9333ea", "#f59e0b"];

function ReportsPage() {
  const { summary, categoryReport, message, loading } = useReports();

  if (loading) {
    return <p>Loading reports...</p>;
  }

  return (
    <>
      <h1>Reports</h1>

      {message && <p className="error">{message}</p>}

      {/* keep your existing summary cards, bar chart, pie chart, and list here */}
    </>
  );
}

export default ReportsPage;
```

Do not delete your chart JSX. Just remove the old `useState`, `useEffect`, and API calls from `ReportsPage.jsx`.

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run React:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-37-react-custom-hooks"
npm install
npm run dev
```

Test:

- Login
- Navigation still works
- `/reports` still shows charts
- Logout redirects to login
- Protected pages still block logged-out users

Commit:

```powershell
git add .
git commit -m "Day 37 refactor React app with custom hooks"
```

Day 37 is complete when you can explain: custom hooks let you move reusable state and side-effect logic out of page components.
