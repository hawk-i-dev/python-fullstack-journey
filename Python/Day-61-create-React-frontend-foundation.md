Day 61 = React frontend foundation for portfolio project.

Today we create the real frontend for `expense-manager-saas`.

Stack:

```text
React
TypeScript
Vite
React Router
FastAPI backend connection
```

## 1. Check Node version

Vite currently requires modern Node. First check:

```powershell
node --version
npm --version
```

If Node is old, install latest LTS from:

```text
https://nodejs.org
```

## 2. Create frontend app

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

npm create vite@latest frontend -- --template react-ts

cd frontend

npm install
npm install react-router
```

If it says folder already exists, choose:

```text
Ignore files and continue
```

Only do that if your `frontend` folder is empty.

## 3. Create frontend environment file

Inside `frontend/`, create `.env`:

```env
VITE_API_BASE_URL=http://127.0.0.1:8000
```

Also create `.env.example`:

```env
VITE_API_BASE_URL=http://127.0.0.1:8000
```

Important: frontend env variables must start with `VITE_`.

Never put database password or JWT secret in frontend `.env`.

## 4. Add CORS in backend

Open backend file:

```text
backend/app/main.py
```

Add this import:

```python
from fastapi.middleware.cors import CORSMiddleware
```

After this line:

```python
app = FastAPI(title="Expense Manager SaaS API")
```

add:

```python
frontend_origins = [
    "http://localhost:5173",
    "http://127.0.0.1:5173",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=frontend_origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    allow_headers=["Authorization", "Content-Type"],
)
```

This allows React running on port `5173` to call FastAPI running on port `8000`.

## 5. Create frontend folders

Inside `frontend/src/`, create:

```text
api/
components/
layouts/
pages/
types/
```

Final structure should become:

```text
frontend/src/
├── api/
│   └── client.ts
├── components/
│   └── PageCard.tsx
├── layouts/
│   └── AppLayout.tsx
├── pages/
│   ├── CategoriesPage.tsx
│   ├── DashboardPage.tsx
│   ├── ExpensesPage.tsx
│   ├── ExportsPage.tsx
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   └── ReportsPage.tsx
├── types/
│   └── api.ts
├── App.tsx
├── index.css
└── main.tsx
```

## 6. Create `src/api/client.ts`

```ts
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? "http://127.0.0.1:8000";

export class ApiError extends Error {
  status: number;
  data: unknown;

  constructor(status: number, data: unknown) {
    super(`API request failed with status ${status}`);
    this.status = status;
    this.data = data;
  }
}

type ApiRequestOptions = {
  method?: string;
  body?: unknown;
  token?: string | null;
  headers?: HeadersInit;
};

export async function apiRequest<T>(
  path: string,
  options: ApiRequestOptions = {},
): Promise<T> {
  const headers = new Headers(options.headers);

  if (options.body !== undefined && !(options.body instanceof FormData)) {
    headers.set("Content-Type", "application/json");
  }

  if (options.token) {
    headers.set("Authorization", `Bearer ${options.token}`);
  }

  const response = await fetch(`${API_BASE_URL}${path}`, {
    method: options.method ?? "GET",
    headers,
    body:
      options.body instanceof FormData
        ? options.body
        : options.body !== undefined
          ? JSON.stringify(options.body)
          : undefined,
  });

  const contentType = response.headers.get("content-type");
  const data = contentType?.includes("application/json")
    ? await response.json()
    : await response.text();

  if (!response.ok) {
    throw new ApiError(response.status, data);
  }

  return data as T;
}

export { API_BASE_URL };
```

## 7. Create `src/types/api.ts`

```ts
export type UserResponse = {
  id: number;
  username: string;
  email: string;
  is_active: boolean;
};

export type TokenResponse = {
  access_token: string;
  token_type: "bearer";
};

export type CategoryResponse = {
  id: number;
  name: string;
};

export type ExpenseResponse = {
  id: number;
  title: string;
  amount: string;
  category_id: number;
  user_id: number;
  created_at: string;
  category?: CategoryResponse | null;
};

export type ExpenseSummaryResponse = {
  total_expenses: number;
  total_amount: string;
  average_amount: string;
};

export type CategoryReportResponse = {
  category_id: number;
  category_name: string;
  expense_count: number;
  total_amount: string;
  average_amount: string;
};

export type MonthlyReportResponse = {
  month: string;
  expense_count: number;
  total_amount: string;
};

export type DashboardReportResponse = {
  total_expenses: number;
  total_amount: string;
  average_amount: string;
  current_month_total: string;
  top_category: string | null;
  top_category_amount: string;
};
```

## 8. Create `src/components/PageCard.tsx`

```tsx
type PageCardProps = {
  title: string;
  description: string;
};

export function PageCard({ title, description }: PageCardProps) {
  return (
    <section className="page-card">
      <h1>{title}</h1>
      <p>{description}</p>
    </section>
  );
}
```

## 9. Create `src/layouts/AppLayout.tsx`

```tsx
import { NavLink, Outlet } from "react-router";

const navItems = [
  { to: "/dashboard", label: "Dashboard" },
  { to: "/expenses", label: "Expenses" },
  { to: "/categories", label: "Categories" },
  { to: "/reports", label: "Reports" },
  { to: "/exports", label: "Exports" },
  { to: "/login", label: "Login" },
  { to: "/register", label: "Register" },
];

export function AppLayout() {
  return (
    <div className="app-shell">
      <aside className="sidebar">
        <h2>Expense Manager</h2>

        <nav>
          {navItems.map((item) => (
            <NavLink
              key={item.to}
              to={item.to}
              className={({ isActive }) => (isActive ? "nav-link active" : "nav-link")}
            >
              {item.label}
            </NavLink>
          ))}
        </nav>
      </aside>

      <main className="main-content">
        <Outlet />
      </main>
    </div>
  );
}
```

## 10. Create pages

`src/pages/DashboardPage.tsx`

```tsx
import { useEffect, useState } from "react";

import { apiRequest, API_BASE_URL } from "../api/client";

type HealthResponse = {
  status: string;
  service: string;
};

export function DashboardPage() {
  const [health, setHealth] = useState<"checking" | "online" | "offline">("checking");

  useEffect(() => {
    let ignore = false;

    apiRequest<HealthResponse>("/health")
      .then(() => {
        if (!ignore) setHealth("online");
      })
      .catch(() => {
        if (!ignore) setHealth("offline");
      });

    return () => {
      ignore = true;
    };
  }, []);

  return (
    <section className="page-card">
      <h1>Dashboard</h1>
      <p>Your portfolio SaaS frontend is connected to the backend.</p>

      <div className={`status-box ${health}`}>
        <strong>Backend:</strong> {health}
      </div>

      <p className="muted">API URL: {API_BASE_URL}</p>
    </section>
  );
}
```

`src/pages/LoginPage.tsx`

```tsx
import { PageCard } from "../components/PageCard";

export function LoginPage() {
  return <PageCard title="Login" description="Day 62 will implement login with JWT." />;
}
```

`src/pages/RegisterPage.tsx`

```tsx
import { PageCard } from "../components/PageCard";

export function RegisterPage() {
  return <PageCard title="Register" description="Day 62 will implement user registration." />;
}
```

`src/pages/ExpensesPage.tsx`

```tsx
import { PageCard } from "../components/PageCard";

export function ExpensesPage() {
  return <PageCard title="Expenses" description="Day 63 will connect expenses CRUD." />;
}
```

`src/pages/CategoriesPage.tsx`

```tsx
import { PageCard } from "../components/PageCard";

export function CategoriesPage() {
  return <PageCard title="Categories" description="Day 64 will connect categories CRUD." />;
}
```

`src/pages/ReportsPage.tsx`

```tsx
import { PageCard } from "../components/PageCard";

export function ReportsPage() {
  return <PageCard title="Reports" description="Day 65 will show reports and dashboard data." />;
}
```

`src/pages/ExportsPage.tsx`

```tsx
import { PageCard } from "../components/PageCard";

export function ExportsPage() {
  return <PageCard title="Exports" description="Day 66 will add CSV download buttons." />;
}
```

## 11. Replace `src/App.tsx`

```tsx
import { BrowserRouter, Navigate, Route, Routes } from "react-router";

import { AppLayout } from "./layouts/AppLayout";
import { CategoriesPage } from "./pages/CategoriesPage";
import { DashboardPage } from "./pages/DashboardPage";
import { ExpensesPage } from "./pages/ExpensesPage";
import { ExportsPage } from "./pages/ExportsPage";
import { LoginPage } from "./pages/LoginPage";
import { RegisterPage } from "./pages/RegisterPage";
import { ReportsPage } from "./pages/ReportsPage";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route element={<AppLayout />}>
          <Route index element={<Navigate to="/dashboard" replace />} />
          <Route path="/dashboard" element={<DashboardPage />} />
          <Route path="/expenses" element={<ExpensesPage />} />
          <Route path="/categories" element={<CategoriesPage />} />
          <Route path="/reports" element={<ReportsPage />} />
          <Route path="/exports" element={<ExportsPage />} />
          <Route path="/login" element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

## 12. Replace `src/index.css`

```css
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family:
    Inter, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: #f3f4f6;
  color: #111827;
}

a {
  color: inherit;
  text-decoration: none;
}

.app-shell {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: 260px;
  padding: 24px;
  background: #111827;
  color: white;
}

.sidebar h2 {
  margin: 0 0 24px;
  font-size: 22px;
}

.sidebar nav {
  display: grid;
  gap: 8px;
}

.nav-link {
  padding: 10px 12px;
  border-radius: 8px;
  color: #d1d5db;
}

.nav-link:hover,
.nav-link.active {
  background: #2563eb;
  color: white;
}

.main-content {
  flex: 1;
  padding: 32px;
}

.page-card {
  max-width: 900px;
  padding: 28px;
  border-radius: 16px;
  background: white;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
}

.page-card h1 {
  margin: 0 0 12px;
  font-size: 32px;
}

.page-card p {
  margin: 0 0 16px;
  color: #4b5563;
}

.status-box {
  width: fit-content;
  margin-top: 20px;
  padding: 12px 14px;
  border-radius: 10px;
  font-weight: 500;
}

.status-box.checking {
  background: #fef3c7;
  color: #92400e;
}

.status-box.online {
  background: #dcfce7;
  color: #166534;
}

.status-box.offline {
  background: #fee2e2;
  color: #991b1b;
}

.muted {
  margin-top: 16px;
  font-size: 14px;
  color: #6b7280;
}
```

## 13. Run backend and frontend

Terminal 1:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Terminal 2:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run dev
```

Open:

```text
http://localhost:5173
```

Expected:

```text
Dashboard page opens
Sidebar navigation works
Backend status shows online
```

If backend status shows `offline`, check:

```text
Backend running on http://127.0.0.1:8000
Frontend .env has VITE_API_BASE_URL=http://127.0.0.1:8000
CORS middleware added in backend/app/main.py
```

## 14. Build check

```powershell
npm run build
```

If build passes, commit:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 61 create React frontend foundation"
```

Senior concept today: frontend should not directly scatter `fetch()` everywhere. A central API client gives you one place to handle base URL, headers, tokens, JSON parsing, and errors.

Sources checked: [Vite getting started](https://vite.dev/guide/), [Vite env variables](https://main.vite.dev/guide/env-and-mode), [React Router API](https://api.reactrouter.com/v7/modules/react-router.html), [FastAPI CORS](https://fastapi.tiangolo.com/tutorial/cors/).
