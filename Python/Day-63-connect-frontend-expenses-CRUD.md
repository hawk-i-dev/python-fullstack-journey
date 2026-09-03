Day 63 = Frontend Expenses CRUD.

Today we connect the React UI to these backend APIs:

```text
GET    /expenses
GET    /expenses/summary
POST   /expenses
PUT    /expenses/{expense_id}
DELETE /expenses/{expense_id}
GET    /categories
```

## 1. Go to frontend

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run dev
```

Backend should also be running:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

## 2. Create `src/api/categories.ts`

```ts
import { apiRequest } from "./client";
import type { CategoryResponse } from "../types/api";

export function listCategories(token: string) {
  return apiRequest<CategoryResponse[]>("/categories", { token });
}
```

## 3. Create `src/api/expenses.ts`

```ts
import { apiRequest } from "./client";
import type { ExpenseResponse, ExpenseSummaryResponse } from "../types/api";

export type ExpenseFilters = {
  q?: string;
  category_id?: string;
  start_date?: string;
  end_date?: string;
  min_amount?: string;
  max_amount?: string;
  sort_by?: "created_at" | "amount" | "title";
  sort_order?: "asc" | "desc";
};

export type ExpensePayload = {
  title: string;
  amount: string;
  category_id: number;
};

function appendParam(params: URLSearchParams, key: string, value: string | undefined) {
  if (value && value.trim()) {
    params.set(key, value.trim());
  }
}

function buildExpenseQuery(filters: ExpenseFilters) {
  const params = new URLSearchParams();

  appendParam(params, "q", filters.q);
  appendParam(params, "category_id", filters.category_id);
  appendParam(params, "start_date", filters.start_date);
  appendParam(params, "end_date", filters.end_date);
  appendParam(params, "min_amount", filters.min_amount);
  appendParam(params, "max_amount", filters.max_amount);
  appendParam(params, "sort_by", filters.sort_by);
  appendParam(params, "sort_order", filters.sort_order);

  const query = params.toString();

  return query ? `?${query}` : "";
}

export function listExpenses(token: string, filters: ExpenseFilters = {}) {
  return apiRequest<ExpenseResponse[]>(`/expenses${buildExpenseQuery(filters)}`, {
    token,
  });
}

export function getExpenseSummary(token: string, filters: ExpenseFilters = {}) {
  return apiRequest<ExpenseSummaryResponse>(`/expenses/summary${buildExpenseQuery(filters)}`, {
    token,
  });
}

export function createExpense(token: string, payload: ExpensePayload) {
  return apiRequest<ExpenseResponse>("/expenses", {
    method: "POST",
    token,
    body: payload,
  });
}

export function updateExpense(token: string, expenseId: number, payload: ExpensePayload) {
  return apiRequest<ExpenseResponse>(`/expenses/${expenseId}`, {
    method: "PUT",
    token,
    body: payload,
  });
}

export function deleteExpense(token: string, expenseId: number) {
  return apiRequest<{ message: string }>(`/expenses/${expenseId}`, {
    method: "DELETE",
    token,
  });
}
```

## 4. Replace `src/pages/ExpensesPage.tsx`

```tsx
import { type FormEvent, useEffect, useState } from "react";

import { listCategories } from "../api/categories";
import { getApiErrorMessage } from "../api/errors";
import {
  createExpense,
  deleteExpense,
  getExpenseSummary,
  listExpenses,
  updateExpense,
  type ExpenseFilters,
} from "../api/expenses";
import { useAuth } from "../auth/AuthContext";
import type {
  CategoryResponse,
  ExpenseResponse,
  ExpenseSummaryResponse,
} from "../types/api";

const emptyForm = {
  title: "",
  amount: "",
  category_id: "",
};

const defaultFilters: ExpenseFilters = {
  sort_by: "created_at",
  sort_order: "desc",
};

function formatMoney(value: string) {
  return Number(value).toLocaleString("en-IN", {
    style: "currency",
    currency: "INR",
  });
}

function formatDate(value: string) {
  return new Date(value).toLocaleDateString("en-IN", {
    year: "numeric",
    month: "short",
    day: "2-digit",
  });
}

export function ExpensesPage() {
  const { token } = useAuth();

  const [categories, setCategories] = useState<CategoryResponse[]>([]);
  const [expenses, setExpenses] = useState<ExpenseResponse[]>([]);
  const [summary, setSummary] = useState<ExpenseSummaryResponse | null>(null);

  const [filters, setFilters] = useState<ExpenseFilters>(defaultFilters);
  const [appliedFilters, setAppliedFilters] = useState<ExpenseFilters>(defaultFilters);

  const [form, setForm] = useState(emptyForm);
  const [editingExpenseId, setEditingExpenseId] = useState<number | null>(null);

  const [loading, setLoading] = useState(true);
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [reloadKey, setReloadKey] = useState(0);

  useEffect(() => {
    if (!token) return;

    let ignore = false;

    async function loadExpensesPage() {
      setLoading(true);
      setError(null);

      try {
        const [categoryData, expenseData, summaryData] = await Promise.all([
          listCategories(token),
          listExpenses(token, appliedFilters),
          getExpenseSummary(token, appliedFilters),
        ]);

        if (!ignore) {
          setCategories(categoryData);
          setExpenses(expenseData);
          setSummary(summaryData);
        }
      } catch (err) {
        if (!ignore) {
          setError(getApiErrorMessage(err, "Failed to load expenses"));
        }
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }

    void loadExpensesPage();

    return () => {
      ignore = true;
    };
  }, [token, appliedFilters, reloadKey]);

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    if (!token) return;

    setSubmitting(true);
    setError(null);

    try {
      const payload = {
        title: form.title.trim(),
        amount: form.amount,
        category_id: Number(form.category_id),
      };

      if (editingExpenseId) {
        await updateExpense(token, editingExpenseId, payload);
      } else {
        await createExpense(token, payload);
      }

      setForm(emptyForm);
      setEditingExpenseId(null);
      setReloadKey((current) => current + 1);
    } catch (err) {
      setError(getApiErrorMessage(err, "Failed to save expense"));
    } finally {
      setSubmitting(false);
    }
  }

  function handleApplyFilters(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();
    setAppliedFilters({ ...filters });
  }

  function handleResetFilters() {
    setFilters(defaultFilters);
    setAppliedFilters(defaultFilters);
  }

  function startEdit(expense: ExpenseResponse) {
    setEditingExpenseId(expense.id);
    setForm({
      title: expense.title,
      amount: String(expense.amount),
      category_id: String(expense.category_id),
    });

    window.scrollTo({ top: 0, behavior: "smooth" });
  }

  function cancelEdit() {
    setEditingExpenseId(null);
    setForm(emptyForm);
  }

  async function handleDelete(expenseId: number) {
    if (!token) return;

    const confirmed = window.confirm("Delete this expense?");

    if (!confirmed) return;

    try {
      await deleteExpense(token, expenseId);
      setReloadKey((current) => current + 1);
    } catch (err) {
      setError(getApiErrorMessage(err, "Failed to delete expense"));
    }
  }

  return (
    <section className="page-stack">
      <div className="page-card">
        <h1>Expenses</h1>
        <p>Create, update, delete, search, filter, and summarize your expenses.</p>

        {error && <div className="error-alert">{error}</div>}

        {categories.length === 0 && !loading && (
          <div className="warning-alert">
            No categories found. Create one from Swagger for now. Day 64 will add the
            Categories UI.
          </div>
        )}

        <form className="form two-column-form" onSubmit={handleSubmit}>
          <div className="form-group">
            <label htmlFor="title">Title</label>
            <input
              id="title"
              value={form.title}
              onChange={(event) => setForm({ ...form, title: event.target.value })}
              placeholder="Breakfast"
              required
            />
          </div>

          <div className="form-group">
            <label htmlFor="amount">Amount</label>
            <input
              id="amount"
              type="number"
              min="0.01"
              step="0.01"
              value={form.amount}
              onChange={(event) => setForm({ ...form, amount: event.target.value })}
              placeholder="120.50"
              required
            />
          </div>

          <div className="form-group">
            <label htmlFor="category">Category</label>
            <select
              id="category"
              value={form.category_id}
              onChange={(event) => setForm({ ...form, category_id: event.target.value })}
              required
            >
              <option value="">Select category</option>
              {categories.map((category) => (
                <option key={category.id} value={category.id}>
                  {category.name}
                </option>
              ))}
            </select>
          </div>

          <div className="form-actions">
            <button className="primary-button" disabled={submitting || categories.length === 0}>
              {submitting
                ? "Saving..."
                : editingExpenseId
                  ? "Update expense"
                  : "Add expense"}
            </button>

            {editingExpenseId && (
              <button type="button" className="secondary-button" onClick={cancelEdit}>
                Cancel edit
              </button>
            )}
          </div>
        </form>
      </div>

      <div className="summary-grid">
        <div className="summary-card">
          <span>Total expenses</span>
          <strong>{summary?.total_expenses ?? 0}</strong>
        </div>

        <div className="summary-card">
          <span>Total amount</span>
          <strong>{summary ? formatMoney(summary.total_amount) : "₹0.00"}</strong>
        </div>

        <div className="summary-card">
          <span>Average amount</span>
          <strong>{summary ? formatMoney(summary.average_amount) : "₹0.00"}</strong>
        </div>
      </div>

      <div className="page-card">
        <h2>Filters</h2>

        <form className="filters-grid" onSubmit={handleApplyFilters}>
          <div className="form-group">
            <label htmlFor="q">Search</label>
            <input
              id="q"
              value={filters.q ?? ""}
              onChange={(event) => setFilters({ ...filters, q: event.target.value })}
              placeholder="Search title"
            />
          </div>

          <div className="form-group">
            <label htmlFor="filterCategory">Category</label>
            <select
              id="filterCategory"
              value={filters.category_id ?? ""}
              onChange={(event) =>
                setFilters({
                  ...filters,
                  category_id: event.target.value || undefined,
                })
              }
            >
              <option value="">All categories</option>
              {categories.map((category) => (
                <option key={category.id} value={category.id}>
                  {category.name}
                </option>
              ))}
            </select>
          </div>

          <div className="form-group">
            <label htmlFor="startDate">Start date</label>
            <input
              id="startDate"
              type="date"
              value={filters.start_date ?? ""}
              onChange={(event) =>
                setFilters({ ...filters, start_date: event.target.value || undefined })
              }
            />
          </div>

          <div className="form-group">
            <label htmlFor="endDate">End date</label>
            <input
              id="endDate"
              type="date"
              value={filters.end_date ?? ""}
              onChange={(event) =>
                setFilters({ ...filters, end_date: event.target.value || undefined })
              }
            />
          </div>

          <div className="form-group">
            <label htmlFor="minAmount">Min amount</label>
            <input
              id="minAmount"
              type="number"
              min="0"
              step="0.01"
              value={filters.min_amount ?? ""}
              onChange={(event) =>
                setFilters({ ...filters, min_amount: event.target.value || undefined })
              }
            />
          </div>

          <div className="form-group">
            <label htmlFor="maxAmount">Max amount</label>
            <input
              id="maxAmount"
              type="number"
              min="0"
              step="0.01"
              value={filters.max_amount ?? ""}
              onChange={(event) =>
                setFilters({ ...filters, max_amount: event.target.value || undefined })
              }
            />
          </div>

          <div className="form-group">
            <label htmlFor="sortBy">Sort by</label>
            <select
              id="sortBy"
              value={filters.sort_by ?? "created_at"}
              onChange={(event) =>
                setFilters({
                  ...filters,
                  sort_by: event.target.value as ExpenseFilters["sort_by"],
                })
              }
            >
              <option value="created_at">Created date</option>
              <option value="amount">Amount</option>
              <option value="title">Title</option>
            </select>
          </div>

          <div className="form-group">
            <label htmlFor="sortOrder">Sort order</label>
            <select
              id="sortOrder"
              value={filters.sort_order ?? "desc"}
              onChange={(event) =>
                setFilters({
                  ...filters,
                  sort_order: event.target.value as ExpenseFilters["sort_order"],
                })
              }
            >
              <option value="desc">Descending</option>
              <option value="asc">Ascending</option>
            </select>
          </div>

          <div className="filter-actions">
            <button className="primary-button">Apply filters</button>
            <button type="button" className="secondary-button" onClick={handleResetFilters}>
              Reset
            </button>
          </div>
        </form>
      </div>

      <div className="table-card">
        <h2>Expense list</h2>

        {loading ? (
          <p className="muted">Loading expenses...</p>
        ) : expenses.length === 0 ? (
          <p className="muted">No expenses found.</p>
        ) : (
          <div className="table-wrapper">
            <table>
              <thead>
                <tr>
                  <th>Title</th>
                  <th>Category</th>
                  <th>Amount</th>
                  <th>Date</th>
                  <th>Actions</th>
                </tr>
              </thead>

              <tbody>
                {expenses.map((expense) => (
                  <tr key={expense.id}>
                    <td>{expense.title}</td>
                    <td>{expense.category?.name ?? "Unknown"}</td>
                    <td>{formatMoney(expense.amount)}</td>
                    <td>{formatDate(expense.created_at)}</td>
                    <td>
                      <div className="row-actions">
                        <button className="small-button" onClick={() => startEdit(expense)}>
                          Edit
                        </button>
                        <button
                          className="small-button danger"
                          onClick={() => void handleDelete(expense.id)}
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

## 5. Add CSS to `src/index.css`

Append:

```css
.page-stack {
  display: grid;
  gap: 24px;
}

.two-column-form {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

.form-group select {
  padding: 11px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  background: white;
  font: inherit;
}

.form-actions,
.filter-actions {
  display: flex;
  align-items: end;
  gap: 12px;
}

.secondary-button {
  width: fit-content;
  padding: 11px 16px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  background: white;
  color: #111827;
  font-weight: 700;
  cursor: pointer;
}

.secondary-button:hover {
  background: #f3f4f6;
}

.warning-alert {
  max-width: 680px;
  margin-top: 18px;
  padding: 12px 14px;
  border-radius: 8px;
  background: #fef3c7;
  color: #92400e;
  font-weight: 600;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 16px;
  max-width: 900px;
}

.summary-card {
  padding: 20px;
  border-radius: 16px;
  background: white;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
}

.summary-card span {
  display: block;
  margin-bottom: 8px;
  color: #6b7280;
  font-size: 14px;
}

.summary-card strong {
  font-size: 26px;
}

.filters-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 16px;
  margin-top: 20px;
}

.table-card {
  max-width: 1100px;
  padding: 24px;
  border-radius: 16px;
  background: white;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
}

.table-card h2,
.page-card h2 {
  margin-top: 0;
}

.table-wrapper {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 14px 12px;
  border-bottom: 1px solid #e5e7eb;
  text-align: left;
}

th {
  color: #374151;
  font-size: 14px;
}

.row-actions {
  display: flex;
  gap: 8px;
}

.small-button {
  padding: 7px 10px;
  border: 0;
  border-radius: 7px;
  background: #e5e7eb;
  color: #111827;
  font-weight: 700;
  cursor: pointer;
}

.small-button:hover {
  background: #d1d5db;
}

.small-button.danger {
  background: #fee2e2;
  color: #991b1b;
}

.small-button.danger:hover {
  background: #fecaca;
}

@media (max-width: 900px) {
  .app-shell {
    display: block;
  }

  .sidebar {
    width: 100%;
  }

  .two-column-form,
  .filters-grid,
  .summary-grid {
    grid-template-columns: 1fr;
  }
}
```

## 6. Test manually

Open:

```text
http://localhost:5173/expenses
```

Test this flow:

```text
Login
Go to Expenses
Create expense
See expense in table
Edit expense
Apply search filter
Apply amount filter
Delete expense
Refresh page and confirm data is still correct
```

If category dropdown is empty, create one from Swagger first:

```text
http://127.0.0.1:8000/docs
POST /categories
```

Day 64 will build the Categories UI.

## 7. Build check

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run build
```

## 8. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 63 connect frontend expenses CRUD"
```

Senior concept: frontend CRUD should not hide authorization. Every API request from private pages must send the JWT token. The backend still remains the real security layer, but the frontend must consistently attach:

```text
Authorization: Bearer <token>
```

Sources checked: [React useEffect](https://react.dev/reference/react/useEffect), [React useState](https://react.dev/reference/react/useState), [MDN URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams), [MDN date input](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/date).
