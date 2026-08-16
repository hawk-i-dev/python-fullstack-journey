`Day 30`: React authentication with JWT.

Today you build login/register/logout in React and connect it to your Day 21 FastAPI auth backend. We will not connect protected expenses yet. That is Day 31.

Sources checked: React Router currently imports navigation tools from `react-router`; `useNavigate` is used for programmatic navigation after form submit. `localStorage` persists token data across browser sessions. See [React Router navigation](https://reactrouter.com/start/declarative/navigating), [useNavigate](https://reactrouter.com/api/hooks/useNavigate), and [MDN localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

**Day 30 Goal**

- Register user from React
- Login user from React
- Store JWT token
- Send `Authorization: Bearer <token>`
- Fetch logged-in user using `/auth/me`
- Logout
- Protect frontend pages

Start backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

In another terminal:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
npm create vite@latest day-30-react-auth -- --template react
cd day-30-react-auth
npm install
npm install react-router
code .
```

Create `.env`:

```env
VITE_API_URL=http://127.0.0.1:8000
```

Create structure:

```text
src/
├── api/
│   └── auth.js
├── pages/
│   ├── LoginPage.jsx
│   ├── RegisterPage.jsx
│   └── ProfilePage.jsx
├── App.jsx
├── App.css
└── main.jsx
```

`src/api/auth.js`:

```javascript
const API_URL = import.meta.env.VITE_API_URL;

export function getToken() {
  return localStorage.getItem("access_token");
}

export function saveToken(token) {
  localStorage.setItem("access_token", token);
}

export function logout() {
  localStorage.removeItem("access_token");
}

export async function registerUser(user) {
  const response = await fetch(`${API_URL}/auth/register`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(user),
  });

  if (!response.ok) {
    throw new Error("Registration failed.");
  }

  return response.json();
}

export async function loginUser(username, password) {
  const formData = new URLSearchParams();
  formData.append("username", username);
  formData.append("password", password);

  const response = await fetch(`${API_URL}/auth/login`, {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: formData,
  });

  if (!response.ok) {
    throw new Error("Login failed.");
  }

  return response.json();
}

export async function getCurrentUser() {
  const token = getToken();

  const response = await fetch(`${API_URL}/auth/me`, {
    headers: {
      Authorization: `Bearer ${token}`,
    },
  });

  if (!response.ok) {
    throw new Error("Not authenticated.");
  }

  return response.json();
}
```

`src/main.jsx`:

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

`src/App.jsx`:

```jsx
import { Link, Route, Routes, useNavigate } from "react-router";
import { getToken, logout } from "./api/auth";
import LoginPage from "./pages/LoginPage";
import ProfilePage from "./pages/ProfilePage";
import RegisterPage from "./pages/RegisterPage";
import "./App.css";

function App() {
  const navigate = useNavigate();
  const isLoggedIn = Boolean(getToken());

  function handleLogout() {
    logout();
    navigate("/login");
  }

  return (
    <main className="container">
      <nav className="nav">
        <Link to="/register">Register</Link>
        <Link to="/login">Login</Link>
        <Link to="/profile">Profile</Link>

        {isLoggedIn && <button onClick={handleLogout}>Logout</button>}
      </nav>

      <Routes>
        <Route path="/" element={<LoginPage />} />
        <Route path="/register" element={<RegisterPage />} />
        <Route path="/login" element={<LoginPage />} />
        <Route path="/profile" element={<ProfilePage />} />
      </Routes>
    </main>
  );
}

export default App;
```

`src/pages/RegisterPage.jsx`:

```jsx
import { useState } from "react";
import { useNavigate } from "react-router";
import { registerUser } from "../api/auth";

function RegisterPage() {
  const navigate = useNavigate();
  const [form, setForm] = useState({ username: "", email: "", password: "" });
  const [message, setMessage] = useState("");

  function handleChange(event) {
    setForm({ ...form, [event.target.name]: event.target.value });
  }

  async function handleSubmit(event) {
    event.preventDefault();

    try {
      await registerUser(form);
      navigate("/login");
    } catch (error) {
      setMessage(error.message);
    }
  }

  return (
    <>
      <h1>Register</h1>

      <form onSubmit={handleSubmit}>
        <input name="username" value={form.username} onChange={handleChange} placeholder="Username" />
        <input name="email" value={form.email} onChange={handleChange} placeholder="Email" />
        <input name="password" value={form.password} onChange={handleChange} type="password" placeholder="Password" />
        <button type="submit">Register</button>
      </form>

      {message && <p className="error">{message}</p>}
    </>
  );
}

export default RegisterPage;
```

`src/pages/LoginPage.jsx`:

```jsx
import { useState } from "react";
import { useNavigate } from "react-router";
import { loginUser, saveToken } from "../api/auth";

function LoginPage() {
  const navigate = useNavigate();
  const [form, setForm] = useState({ username: "", password: "" });
  const [message, setMessage] = useState("");

  function handleChange(event) {
    setForm({ ...form, [event.target.name]: event.target.value });
  }

  async function handleSubmit(event) {
    event.preventDefault();

    try {
      const data = await loginUser(form.username, form.password);
      saveToken(data.access_token);
      navigate("/profile");
    } catch (error) {
      setMessage(error.message);
    }
  }

  return (
    <>
      <h1>Login</h1>

      <form onSubmit={handleSubmit}>
        <input name="username" value={form.username} onChange={handleChange} placeholder="Username" />
        <input name="password" value={form.password} onChange={handleChange} type="password" placeholder="Password" />
        <button type="submit">Login</button>
      </form>

      {message && <p className="error">{message}</p>}
    </>
  );
}

export default LoginPage;
```

`src/pages/ProfilePage.jsx`:

```jsx
import { useEffect, useState } from "react";
import { Link } from "react-router";
import { getCurrentUser } from "../api/auth";

function ProfilePage() {
  const [user, setUser] = useState(null);
  const [message, setMessage] = useState("");

  useEffect(() => {
    async function loadUser() {
      try {
        setUser(await getCurrentUser());
      } catch (error) {
        setMessage(error.message);
      }
    }

    loadUser();
  }, []);

  if (message) {
    return (
      <>
        <p className="error">{message}</p>
        <Link to="/login">Login again</Link>
      </>
    );
  }

  if (!user) {
    return <p>Loading profile...</p>;
  }

  return (
    <>
      <h1>Profile</h1>
      <p>Username: {user.username}</p>
      <p>Email: {user.email}</p>
      <p>Active: {user.is_active ? "Yes" : "No"}</p>
    </>
  );
}

export default ProfilePage;
```

`src/App.css`:

```css
body {
  font-family: Arial, sans-serif;
  background: #f4f6f8;
  margin: 0;
}

.container {
  max-width: 720px;
  margin: 40px auto;
  background: white;
  padding: 24px;
  border-radius: 8px;
}

.nav {
  display: flex;
  gap: 16px;
  align-items: center;
  margin-bottom: 24px;
}

form {
  display: grid;
  gap: 12px;
}

input,
button {
  padding: 12px;
  font-size: 16px;
}

button {
  background: #2563eb;
  color: white;
  border: none;
  cursor: pointer;
}

.error {
  color: #dc2626;
}
```

Run React:

```powershell
npm run dev
```

Test:

- Register user
- Login user
- Confirm redirect to profile
- Refresh browser on `/profile`
- Profile should still load because token is in `localStorage`
- Logout
- Try `/profile` again

Commit:

```powershell
git add .
git commit -m "Day 30 add React JWT authentication"
```

Day 30 is complete when you can explain: login returns a JWT, React stores it, and every protected API request must send it in the `Authorization` header.
