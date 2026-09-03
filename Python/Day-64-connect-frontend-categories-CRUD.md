Day 64 = Frontend Categories CRUD.

Today we connect the Categories page to backend:

```text
GET    /categories
POST   /categories
PUT    /categories/{category_id}
DELETE /categories/{category_id}
```

## 1. Start backend and frontend

Backend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Frontend:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run dev
```

## 2. Replace `src/api/categories.ts`

```ts
import { apiRequest } from "./client";
import type { CategoryResponse } from "../types/api";

export type CategoryPayload = {
  name: string;
};

export function listCategories(token: string) {
  return apiRequest<CategoryResponse[]>("/categories", { token });
}

export function createCategory(token: string, payload: CategoryPayload) {
  return apiRequest<CategoryResponse>("/categories", {
    method: "POST",
    token,
    body: payload,
  });
}

export function updateCategory(
  token: string,
  categoryId: number,
  payload: CategoryPayload,
) {
  return apiRequest<CategoryResponse>(`/categories/${categoryId}`, {
    method: "PUT",
    token,
    body: payload,
  });
}

export function deleteCategory(token: string, categoryId: number) {
  return apiRequest<{ message: string }>(`/categories/${categoryId}`, {
    method: "DELETE",
    token,
  });
}
```

## 3. Replace `src/pages/CategoriesPage.tsx`

```tsx
import { type FormEvent, useEffect, useState } from "react";

import {
  createCategory,
  deleteCategory,
  listCategories,
  updateCategory,
} from "../api/categories";
import { getApiErrorMessage } from "../api/errors";
import { useAuth } from "../auth/AuthContext";
import type { CategoryResponse } from "../types/api";

export function CategoriesPage() {
  const { token } = useAuth();

  const [categories, setCategories] = useState<CategoryResponse[]>([]);
  const [name, setName] = useState("");
  const [editingCategoryId, setEditingCategoryId] = useState<number | null>(null);

  const [loading, setLoading] = useState(true);
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState<string | null>(null);
  const [reloadKey, setReloadKey] = useState(0);

  useEffect(() => {
    if (!token) return;

    const authToken = token;
    let ignore = false;

    async function loadCategories() {
      setLoading(true);
      setError(null);

      try {
        const data = await listCategories(authToken);

        if (!ignore) {
          setCategories(data);
        }
      } catch (err) {
        if (!ignore) {
          setError(getApiErrorMessage(err, "Failed to load categories"));
        }
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }

    void loadCategories();

    return () => {
      ignore = true;
    };
  }, [token, reloadKey]);

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    if (!token) return;

    const trimmedName = name.trim();

    if (!trimmedName) {
      setError("Category name is required");
      return;
    }

    setSubmitting(true);
    setError(null);
    setSuccess(null);

    try {
      if (editingCategoryId) {
        await updateCategory(token, editingCategoryId, { name: trimmedName });
        setSuccess("Category updated successfully");
      } else {
        await createCategory(token, { name: trimmedName });
        setSuccess("Category created successfully");
      }

      setName("");
      setEditingCategoryId(null);
      setReloadKey((current) => current + 1);
    } catch (err) {
      setError(getApiErrorMessage(err, "Failed to save category"));
    } finally {
      setSubmitting(false);
    }
  }

  function startEdit(category: CategoryResponse) {
    setEditingCategoryId(category.id);
    setName(category.name);
    setError(null);
    setSuccess(null);
    window.scrollTo({ top: 0, behavior: "smooth" });
  }

  function cancelEdit() {
    setEditingCategoryId(null);
    setName("");
    setError(null);
    setSuccess(null);
  }

  async function handleDelete(category: CategoryResponse) {
    if (!token) return;

    const confirmed = window.confirm(
      `Delete category "${category.name}"? If it has expenses, backend will block deletion.`,
    );

    if (!confirmed) return;

    setError(null);
    setSuccess(null);

    try {
      await deleteCategory(token, category.id);

      if (editingCategoryId === category.id) {
        cancelEdit();
      }

      setSuccess("Category deleted successfully");
      setReloadKey((current) => current + 1);
    } catch (err) {
      setError(
        getApiErrorMessage(
          err,
          "Failed to delete category. Delete or move expenses first.",
        ),
      );
    }
  }

  return (
    <section className="page-stack">
      <div className="page-card">
        <h1>Categories</h1>
        <p>
          Manage your own expense categories. Categories are private per user.
        </p>

        {error && <div className="error-alert">{error}</div>}
        {success && <div className="success-alert">{success}</div>}

        <form className="form category-form" onSubmit={handleSubmit}>
          <div className="form-group">
            <label htmlFor="categoryName">Category name</label>
            <input
              id="categoryName"
              value={name}
              onChange={(event) => setName(event.target.value)}
              placeholder="Food"
              maxLength={50}
              required
            />
          </div>

          <div className="form-actions">
            <button className="primary-button" disabled={submitting}>
              {submitting
                ? "Saving..."
                : editingCategoryId
                  ? "Update category"
                  : "Add category"}
            </button>

            {editingCategoryId && (
              <button type="button" className="secondary-button" onClick={cancelEdit}>
                Cancel edit
              </button>
            )}
          </div>
        </form>
      </div>

      <div className="summary-grid">
        <div className="summary-card">
          <span>Total categories</span>
          <strong>{categories.length}</strong>
        </div>
      </div>

      <div className="table-card">
        <h2>Category list</h2>

        {loading ? (
          <p className="muted">Loading categories...</p>
        ) : categories.length === 0 ? (
          <p className="muted">No categories found. Create your first category.</p>
        ) : (
          <div className="table-wrapper">
            <table>
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Name</th>
                  <th>Status</th>
                  <th>Actions</th>
                </tr>
              </thead>

              <tbody>
                {categories.map((category) => (
                  <tr key={category.id}>
                    <td>{category.id}</td>
                    <td className="category-name-cell">{category.name}</td>
                    <td>
                      {editingCategoryId === category.id ? (
                        <span className="status-pill editing">Editing</span>
                      ) : (
                        <span className="status-pill ready">Ready</span>
                      )}
                    </td>
                    <td>
                      <div className="row-actions">
                        <button
                          type="button"
                          className="small-button"
                          onClick={() => startEdit(category)}
                        >
                          Edit
                        </button>

                        <button
                          type="button"
                          className="small-button danger"
                          onClick={() => void handleDelete(category)}
                        >
                          Delete
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        )}
      </div>
    </section>
  );
}
```

## 4. Add CSS to `src/index.css`

Append:

```css
.category-form {
  max-width: 560px;
}

.category-name-cell {
  font-weight: 700;
}

.status-pill {
  display: inline-block;
  padding: 5px 9px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 700;
}

.status-pill.ready {
  background: #dcfce7;
  color: #166534;
}

.status-pill.editing {
  background: #dbeafe;
  color: #1d4ed8;
}
```

## 5. Test in browser

Open:

```text
http://localhost:5173/categories
```

Test:

```text
Create Food
Create Travel
Create Bills
Try duplicate Food
Edit Bills to Utilities
Delete Travel
Try deleting Food after creating expenses under Food
```

Expected behavior:

```text
Duplicate category shows error
Category list refreshes after create/update/delete
Category with existing expenses cannot be deleted
Only logged-in user's categories are visible
```

## 6. Build check

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run build
```

## 7. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 64 connect frontend categories CRUD"
```

Senior concept: categories look simple, but they are business data. They must be user-owned, duplicate-protected per user, and protected from unsafe deletion when expenses depend on them.

Sources checked: [React useEffect](https://react.dev/reference/react/useEffect), [MDN select element](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/select), [FastAPI error handling](https://fastapi.tiangolo.com/tutorial/handling-errors/).
