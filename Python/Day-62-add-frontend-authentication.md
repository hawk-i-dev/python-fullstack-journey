Day 62 = Frontend Auth: Register, Login, JWT storage, Logout, Protected Routes.

Regular track: Windows + PostgreSQL.

## 1. Go to frontend

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm install react-router
```

Backend should also be running:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

## 2. Create folders

Inside `frontend/src/`, create:

```text
auth/
api/
```

## 3. Create `src/api/errors.ts`

```ts
import { ApiError } from "./client";

type ValidationError = {
  msg?: string;
};

type ErrorBody = {
  detail?: string | ValidationError[];
};

export function getApiErrorMessage(error: unknown, fallback = "Something went wrong") {
  if (error instanceof ApiError) {
    if (typeof error.data === "string" && error.data.trim()) {
      return error.data;
    }

    if (error.data && typeof error.data === "object" && "detail" in error.data) {
      const detail = (error.data as ErrorBody).detail;

      if (typeof detail === "string") {
        return detail;
      }

      if (Array.isArray(detail)) {
        return detail.map((item) => item.msg ?? "Validation error").join(", ");
      }
    }

    return `Request failed with status ${error.status}`;
  }

  if (error instanceof Error) {
    return error.message;
  }

  return fallback;
}
```

## 4. Create `src/auth/AuthContext.tsx`

```tsx
import {
  createContext,
  useContext,
  useEffect,
  useState,
  type ReactNode,
} from "react";

import { apiRequest } from "../api/client";
import type { TokenResponse, UserResponse } from "../types/api";

const TOKEN_STORAGE_KEY = "expense_manager_token";

type RegisterInput = {
  username: string;
  email: string;
  password: string;
};

type AuthContextValue = {
  token: string | null;
  user: UserResponse | null;
  loading: boolean;
  isAuthenticated: boolean;
  register: (input: RegisterInput) => Promise<UserResponse>;
  login: (username: string, password: string) => Promise<void>;
  logout: () => void;
};

const AuthContext = createContext<AuthContextValue | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [token, setToken] = useState<string | null>(() =>
    localStorage.getItem(TOKEN_STORAGE_KEY),
  );
  const [user, setUser] = useState<UserResponse | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!token) {
      setUser(null);
      setLoading(false);
      return;
    }

    let ignore = false;

    setLoading(true);

    apiRequest<UserResponse>("/auth/me", { token })
      .then((currentUser) => {
        if (!ignore) {
          setUser(currentUser);
        }
      })
      .catch(() => {
        if (!ignore) {
          localStorage.removeItem(TOKEN_STORAGE_KEY);
          setToken(null);
          setUser(null);
        }
      })
      .finally(() => {
        if (!ignore) {
          setLoading(false);
        }
      });

    return () => {
      ignore = true;
    };
  }, [token]);

  async function register(input: RegisterInput) {
    return apiRequest<UserResponse>("/auth/register", {
      method: "POST",
      body: input,
    });
  }

  async function login(username: string, password: string) {
    setLoading(true);

    const formData = new FormData();
    formData.append("username", username);
    formData.append("password", password);

    try {
      const tokenResponse = await apiRequest<TokenResponse>("/auth/login", {
        method: "POST",
        body: formData,
      });

      localStorage.setItem(TOKEN_STORAGE_KEY, tokenResponse.access_token);
      setToken(tokenResponse.access_token);

      const currentUser = await apiRequest<UserResponse>("/auth/me", {
        token: tokenResponse.access_token,
      });

      setUser(currentUser);
    } finally {
      setLoading(false);
    }
  }

  function logout() {
    localStorage.removeItem(TOKEN_STORAGE_KEY);
    setToken(null);
    setUser(null);
    setLoading(false);
  }

  return (
    <AuthContext.Provider
      value={{
        token,
        user,
        loading,
        isAuthenticated: Boolean(token && user),
        register,
        login,
        logout,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const value = useContext(AuthContext);

  if (!value) {
    throw new Error("useAuth must be used inside AuthProvider");
  }

  return value;
}
```

## 5. Create `src/auth/ProtectedRoute.tsx`

```tsx
import { Navigate, Outlet, useLocation } from "react-router";

import { useAuth } from "./AuthContext";

export function ProtectedRoute() {
  const location = useLocation();
  const { isAuthenticated, loading } = useAuth();

  if (loading) {
    return (
      <section className="page-card">
        <h1>Loading</h1>
        <p>Checking your login session...</p>
      </section>
    );
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" replace state={{ from: location }} />;
  }

  return <Outlet />;
}
```

## 6. Replace `src/pages/LoginPage.tsx`

```tsx
import { type FormEvent, useState } from "react";
import { Link, Navigate, useLocation, useNavigate } from "react-router";

import { getApiErrorMessage } from "../api/errors";
import { useAuth } from "../auth/AuthContext";

type LoginLocationState = {
  from?: {
    pathname?: string;
  };
  message?: string;
};

export function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();
  const { login, isAuthenticated } = useAuth();

  const state = location.state as LoginLocationState | null;
  const from = state?.from?.pathname ?? "/dashboard";

  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState<string | null>(null);
  const [submitting, setSubmitting] = useState(false);

  if (isAuthenticated) {
    return <Navigate to={from} replace />;
  }

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    setError(null);
    setSubmitting(true);

    try {
      await login(username.trim(), password);
      navigate(from, { replace: true });
    } catch (err) {
      setError(getApiErrorMessage(err, "Login failed"));
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <section className="page-card">
      <h1>Login</h1>
      <p>Use your username and password to access your expense dashboard.</p>

      {state?.message && <div className="success-alert">{state.message}</div>}
      {error && <div className="error-alert">{error}</div>}

      <form className="form" onSubmit={handleSubmit}>
        <div className="form-group">
          <label htmlFor="username">Username</label>
          <input
            id="username"
            value={username}
            onChange={(event) => setUsername(event.target.value)}
            required
          />
        </div>

        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            value={password}
            onChange={(event) => setPassword(event.target.value)}
            required
          />
        </div>

        <button className="primary-button" disabled={submitting}>
          {submitting ? "Logging in..." : "Login"}
        </button>
      </form>

      <p className="auth-footer">
        New user? <Link to="/register">Create account</Link>
      </p>
    </section>
  );
}
```

## 7. Replace `src/pages/RegisterPage.tsx`

```tsx
import { type FormEvent, useState } from "react";
import { Link, Navigate, useNavigate } from "react-router";

import { getApiErrorMessage } from "../api/errors";
import { useAuth } from "../auth/AuthContext";

export function RegisterPage() {
  const navigate = useNavigate();
  const { register, isAuthenticated } = useAuth();

  const [username, setUsername] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [error, setError] = useState<string | null>(null);
  const [submitting, setSubmitting] = useState(false);

  if (isAuthenticated) {
    return <Navigate to="/dashboard" replace />;
  }

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    setError(null);

    if (password !== confirmPassword) {
      setError("Passwords do not match");
      return;
    }

    setSubmitting(true);

    try {
      await register({
        username: username.trim(),
        email: email.trim(),
        password,
      });

      navigate("/login", {
        replace: true,
        state: {
          message: "Registration successful. Login now.",
        },
      });
    } catch (err) {
      setError(getApiErrorMessage(err, "Registration failed"));
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <section className="page-card">
      <h1>Register</h1>
      <p>Create your account to start tracking expenses.</p>

      {error && <div className="error-alert">{error}</div>}

      <form className="form" onSubmit={handleSubmit}>
        <div className="form-group">
          <label htmlFor="username">Username</label>
          <input
            id="username"
            value={username}
            onChange={(event) => setUsername(event.target.value)}
            minLength={3}
            required
          />
        </div>

        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            value={email}
            onChange={(event) => setEmail(event.target.value)}
            required
          />
        </div>

        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            value={password}
            onChange={(event) => setPassword(event.target.value)}
            minLength={6}
            required
          />
        </div>

        <div className="form-group">
          <label htmlFor="confirmPassword">Confirm password</label>
          <input
            id="confirmPassword"
            type="password"
            value={confirmPassword}
            onChange={(event) => setConfirmPassword(event.target.value)}
            minLength={6}
            required
          />
        </div>

        <button className="primary-button" disabled={submitting}>
          {submitting ? "Creating account..." : "Create account"}
        </button>
      </form>

      <p className="auth-footer">
        Already registered? <Link to="/login">Login</Link>
      </p>
    </section>
  );
}
```

## 8. Replace `src/layouts/AppLayout.tsx`

```tsx
import { NavLink, Outlet, useNavigate } from "react-router";

import { useAuth } from "../auth/AuthContext";

const privateNavItems = [
  { to: "/dashboard", label: "Dashboard" },
  { to: "/expenses", label: "Expenses" },
  { to: "/categories", label: "Categories" },
  { to: "/reports", label: "Reports" },
  { to: "/exports", label: "Exports" },
];

const publicNavItems = [
  { to: "/login", label: "Login" },
  { to: "/register", label: "Register" },
];

export function AppLayout() {
  const navigate = useNavigate();
  const { user, isAuthenticated, logout } = useAuth();

  const navItems = isAuthenticated ? privateNavItems : publicNavItems;

  function handleLogout() {
    logout();
    navigate("/login");
  }

  return (
    <div className="app-shell">
      <aside className="sidebar">
        <h2>Expense Manager</h2>

        {user && (
          <div className="user-panel">
            <span>Signed in as</span>
            <strong>{user.username}</strong>
          </div>
        )}

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

        {isAuthenticated && (
          <button className="logout-button" onClick={handleLogout}>
            Logout
          </button>
        )}
      </aside>

      <main className="main-content">
        <Outlet />
      </main>
    </div>
  );
}
```

## 9. Replace `src/App.tsx`

```tsx
import { BrowserRouter, Navigate, Route, Routes } from "react-router";

import { AuthProvider } from "./auth/AuthContext";
import { ProtectedRoute } from "./auth/ProtectedRoute";
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
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route element={<AppLayout />}>
            <Route index element={<Navigate to="/dashboard" replace />} />
            <Route path="/login" element={<LoginPage />} />
            <Route path="/register" element={<RegisterPage />} />

            <Route element={<ProtectedRoute />}>
              <Route path="/dashboard" element={<DashboardPage />} />
              <Route path="/expenses" element={<ExpensesPage />} />
              <Route path="/categories" element={<CategoriesPage />} />
              <Route path="/reports" element={<ReportsPage />} />
              <Route path="/exports" element={<ExportsPage />} />
            </Route>
          </Route>
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

## 10. Update `src/pages/DashboardPage.tsx`

```tsx
import { useEffect, useState } from "react";

import { apiRequest, API_BASE_URL } from "../api/client";
import { useAuth } from "../auth/AuthContext";

type HealthResponse = {
  status: string;
  service: string;
};

export function DashboardPage() {
  const { user } = useAuth();
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
      <p>Welcome back, {user?.username}.</p>

      <div className={`status-box ${health}`}>
        <strong>Backend:</strong> {health}
      </div>

      <p className="muted">API URL: {API_BASE_URL}</p>
    </section>
  );
}
```

## 11. Add CSS to `src/index.css`

Append this at the bottom:

```css
.user-panel {
  display: grid;
  gap: 4px;
  margin-bottom: 20px;
  padding: 12px;
  border-radius: 10px;
  background: #1f2937;
}

.user-panel span {
  font-size: 12px;
  color: #9ca3af;
}

.user-panel strong {
  color: white;
}

.logout-button {
  width: 100%;
  margin-top: 20px;
  padding: 10px 12px;
  border: 0;
  border-radius: 8px;
  background: #dc2626;
  color: white;
  font-weight: 700;
  cursor: pointer;
}

.logout-button:hover {
  background: #b91c1c;
}

.form {
  display: grid;
  gap: 16px;
  max-width: 420px;
  margin-top: 24px;
}

.form-group {
  display: grid;
  gap: 6px;
}

.form-group label {
  font-weight: 700;
  color: #374151;
}

.form-group input {
  padding: 11px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font: inherit;
}

.form-group input:focus {
  border-color: #2563eb;
  outline: 3px solid rgba(37, 99, 235, 0.15);
}

.primary-button {
  width: fit-content;
  padding: 11px 16px;
  border: 0;
  border-radius: 8px;
  background: #2563eb;
  color: white;
  font-weight: 700;
  cursor: pointer;
}

.primary-button:hover {
  background: #1d4ed8;
}

.primary-button:disabled {
  cursor: not-allowed;
  opacity: 0.7;
}

.error-alert,
.success-alert {
  max-width: 520px;
  margin-top: 18px;
  padding: 12px 14px;
  border-radius: 8px;
  font-weight: 600;
}

.error-alert {
  background: #fee2e2;
  color: #991b1b;
}

.success-alert {
  background: #dcfce7;
  color: #166534;
}

.auth-footer {
  margin-top: 20px;
}

.auth-footer a {
  color: #2563eb;
  font-weight: 700;
}
```

## 12. Test flow

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run frontend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run dev
```

Open:

```text
http://localhost:5173
```

Test:

```text
/register
Create account
Redirects to /login
Login using username + password
Redirects to /dashboard
Sidebar shows username
Private routes work
Logout redirects to /login
Opening /expenses after logout redirects to /login
```

## 13. Common bug

For login, backend expects form data, not JSON.

Correct:

```ts
const formData = new FormData();
formData.append("username", username);
formData.append("password", password);
```

Wrong:

```ts
body: {
  username,
  password,
}
```

If you send JSON to `/auth/login`, FastAPI will usually return `422`.

## 14. Build check

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run build
```

## 15. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 62 add frontend authentication"
```

Senior concept: storing JWT in `localStorage` is acceptable for learning and portfolio basics, but production apps must treat XSS seriously. If malicious JavaScript runs in your frontend, it can read localStorage. Later, we can discuss HttpOnly cookie-based auth.

Sources checked: [React Context](https://react.dev/reference/react/createContext), [React useContext](https://react.dev/reference/react/useContext), [MDN Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API), [FastAPI OAuth2 password flow](https://fastapi.tiangolo.com/tutorial/security/simple-oauth2/).
