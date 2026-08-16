Regular `Day 32`: category management + protected routes in React.

Today you remove the Day 31 dependency on Swagger for creating categories. You’ll also clean up auth protection using a reusable `ProtectedRoute`. React Router supports nested routes through `Outlet`, which is the right fit here. Source: [React Router nested routes](https://reactrouter.com/start/declarative/routing), [Outlet](https://reactrouter.com/api/components/Outlet).

**Day 32 Goal**

- Add `/categories` page
- Create categories from React
- List categories from React
- Protect `/profile`, `/expenses`, `/categories`
- Redirect logged-out users to `/login`

Start from Day 31:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-31-react-protected-expenses .\day-32-react-categories
cd day-32-react-categories
code .
```

Create `src/components/ProtectedRoute.jsx`:

```jsx
import { Navigate, Outlet } from "react-router";
import { getToken } from "../api/auth";

function ProtectedRoute() {
  if (!getToken()) {
    return <Navigate to="/login" replace />;
  }

  return <Outlet />;
}

export default ProtectedRoute;
```

Create `src/pages/CategoriesPage.jsx`:

```jsx
import { useEffect, useState } from "react";
import { createCategory, getCategories } from "../api/expenses";

function CategoriesPage() {
  const [categories, setCategories] = useState([]);
  const [name, setName] = useState("");
  const [message, setMessage] = useState("");

  async function loadCategories() {
    try {
      setCategories(await getCategories());
    } catch (error) {
      setMessage(error.message);
    }
  }

  useEffect(() => {
    loadCategories();
  }, []);

  async function handleSubmit(event) {
    event.preventDefault();

    const category = { name: name.trim() };

    if (!category.name) {
      setMessage("Category name is required.");
      return;
    }

    try {
      await createCategory(category);
      setName("");
      setMessage("");
      loadCategories();
    } catch (error) {
      setMessage(error.message);
    }
  }

  return (
    <>
      <h1>Categories</h1>

      <form onSubmit={handleSubmit}>
        <input
          value={name}
          onChange={(event) => setName(event.target.value)}
          placeholder="Category name"
        />
        <button type="submit">Add Category</button>
      </form>

      {message && <p className="error">{message}</p>}

      <ul>
        {categories.map((category) => (
          <li key={category.id}>{category.name}</li>
        ))}
      </ul>
    </>
  );
}

export default CategoriesPage;
```

Update `src/App.jsx` imports:

```jsx
import { Link, Route, Routes, useNavigate } from "react-router";
import ProtectedRoute from "./components/ProtectedRoute";
import CategoriesPage from "./pages/CategoriesPage";
import ExpensesPage from "./pages/ExpensesPage";
import LoginPage from "./pages/LoginPage";
import ProfilePage from "./pages/ProfilePage";
import RegisterPage from "./pages/RegisterPage";
```

Add nav link:

```jsx
<Link to="/categories">Categories</Link>
```

Update routes:

```jsx
<Routes>
  <Route path="/" element={<LoginPage />} />
  <Route path="/register" element={<RegisterPage />} />
  <Route path="/login" element={<LoginPage />} />

  <Route element={<ProtectedRoute />}>
    <Route path="/profile" element={<ProfilePage />} />
    <Route path="/expenses" element={<ExpensesPage />} />
    <Route path="/categories" element={<CategoriesPage />} />
  </Route>
</Routes>
```

In `src/pages/ExpensesPage.jsx`, remove this old text:

```jsx
<p>
  Need a category? Create it in Swagger first using <code>POST /categories</code>.
</p>
```

Replace with:

```jsx
<p>
  Need a category? Create one from the Categories page.
</p>
```

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run React:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-32-react-categories"
npm install
npm run dev
```

Test:

- Open `/expenses` while logged out: should redirect to `/login`
- Login
- Open `/categories`
- Add `Food`, `Travel`, `Study`
- Open `/expenses`
- Category dropdown should show those categories
- Add expense
- Refresh browser
- Data should remain

Commit:

```powershell
git add .
git commit -m "Day 32 add category page and protected routes"
```

Day 32 is complete when you can explain: `ProtectedRoute` blocks private pages unless a JWT exists, and `Outlet` renders the protected child page after the check passes.
