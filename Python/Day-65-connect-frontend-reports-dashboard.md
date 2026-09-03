Day 65 = Frontend Reports + Dashboard UI.

Today we connect these backend APIs:

```text
GET /reports/dashboard
GET /reports/category-summary
GET /reports/monthly-summary
GET /categories
```

No backend migration needed.

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

## 2. Create `src/utils/formatters.ts`

Create folder if missing:

```text
frontend/src/utils/
```

Add:

```ts
export function formatMoney(value: string | number) {
  const numberValue = typeof value === "number" ? value : Number(value);

  if (Number.isNaN(numberValue)) {
    return "₹0.00";
  }

  return new Intl.NumberFormat("en-IN", {
    style: "currency",
    currency: "INR",
  }).format(numberValue);
}

export function formatNumber(value: number) {
  return new Intl.NumberFormat("en-IN").format(value);
}

export function formatMonth(value: string) {
  const [year, month] = value.split("-").map(Number);

  if (!year || !month) {
    return value;
  }

  return new Intl.DateTimeFormat("en-IN", {
    month: "short",
    year: "numeric",
  }).format(new Date(year, month - 1, 1));
}

export function getBarWidth(value: string | number, maxValue: number) {
  const numberValue = typeof value === "number" ? value : Number(value);

  if (maxValue <= 0 || numberValue <= 0) {
    return "0%";
  }

  return `${Math.max(4, Math.round((numberValue / maxValue) * 100))}%`;
}
```

## 3. Create `src/api/reports.ts`

```ts
import { apiRequest } from "./client";
import type {
  CategoryReportResponse,
  DashboardReportResponse,
  MonthlyReportResponse,
} from "../types/api";

export type ReportFilters = {
  start_date?: string;
  end_date?: string;
  category_id?: string;
};

function appendParam(params: URLSearchParams, key: string, value: string | undefined) {
  if (value && value.trim()) {
    params.set(key, value.trim());
  }
}

function buildCategoryReportQuery(filters: ReportFilters) {
  const params = new URLSearchParams();

  appendParam(params, "start_date", filters.start_date);
  appendParam(params, "end_date", filters.end_date);
  appendParam(params, "category_id", filters.category_id);

  const query = params.toString();

  return query ? `?${query}` : "";
}

function buildMonthlyReportQuery(filters: ReportFilters) {
  const params = new URLSearchParams();

  appendParam(params, "start_date", filters.start_date);
  appendParam(params, "end_date", filters.end_date);

  const query = params.toString();

  return query ? `?${query}` : "";
}

export function getDashboardReport(token: string) {
  return apiRequest<DashboardReportResponse>("/reports/dashboard", { token });
}

export function getCategorySummary(token: string, filters: ReportFilters = {}) {
  return apiRequest<CategoryReportResponse[]>(
    `/reports/category-summary${buildCategoryReportQuery(filters)}`,
    { token },
  );
}

export function getMonthlySummary(token: string, filters: ReportFilters = {}) {
  return apiRequest<MonthlyReportResponse[]>(
    `/reports/monthly-summary${buildMonthlyReportQuery(filters)}`,
    { token },
  );
}
```

## 4. Replace `src/pages/DashboardPage.tsx`

```tsx
import { useEffect, useState } from "react";
import { Link } from "react-router";

import { getApiErrorMessage } from "../api/errors";
import { getDashboardReport } from "../api/reports";
import { useAuth } from "../auth/AuthContext";
import type { DashboardReportResponse } from "../types/api";
import { formatMoney, formatNumber } from "../utils/formatters";

export function DashboardPage() {
  const { token, user } = useAuth();

  const [dashboard, setDashboard] = useState<DashboardReportResponse | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    if (!token) return;

    let ignore = false;

    async function loadDashboard() {
      setLoading(true);
      setError(null);

      try {
        const data = await getDashboardReport(token);

        if (!ignore) {
          setDashboard(data);
        }
      } catch (err) {
        if (!ignore) {
          setError(getApiErrorMessage(err, "Failed to load dashboard"));
        }
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }

    void loadDashboard();

    return () => {
      ignore = true;
    };
  }, [token]);

  return (
    <section className="page-stack">
      <div className="page-card">
        <h1>Dashboard</h1>
        <p>Welcome back, {user?.username}. This is your expense overview.</p>

        {error && <div className="error-alert">{error}</div>}
      </div>

      {loading ? (
        <div className="page-card">
          <p className="muted">Loading dashboard...</p>
        </div>
      ) : (
        <>
          <div className="summary-grid">
            <div className="summary-card">
              <span>Total expenses</span>
              <strong>{formatNumber(dashboard?.total_expenses ?? 0)}</strong>
            </div>

            <div className="summary-card">
              <span>Total amount</span>
              <strong>{formatMoney(dashboard?.total_amount ?? "0")}</strong>
            </div>

            <div className="summary-card">
              <span>Average amount</span>
              <strong>{formatMoney(dashboard?.average_amount ?? "0")}</strong>
            </div>

            <div className="summary-card">
              <span>This month</span>
              <strong>{formatMoney(dashboard?.current_month_total ?? "0")}</strong>
            </div>

            <div className="summary-card">
              <span>Top category</span>
              <strong>{dashboard?.top_category ?? "No data"}</strong>
            </div>

            <div className="summary-card">
              <span>Top category amount</span>
              <strong>{formatMoney(dashboard?.top_category_amount ?? "0")}</strong>
            </div>
          </div>

          <div className="quick-actions">
            <Link className="quick-action-card" to="/expenses">
              Add expense
            </Link>

            <Link className="quick-action-card" to="/categories">
              Manage categories
            </Link>

            <Link className="quick-action-card" to="/reports">
              View reports
            </Link>
          </div>
        </>
      )}
    </section>
  );
}
```

## 5. Replace `src/pages/ReportsPage.tsx`

```tsx
import { type FormEvent, useEffect, useMemo, useState } from "react";

import { listCategories } from "../api/categories";
import { getApiErrorMessage } from "../api/errors";
import {
  getCategorySummary,
  getMonthlySummary,
  type ReportFilters,
} from "../api/reports";
import { useAuth } from "../auth/AuthContext";
import type {
  CategoryReportResponse,
  CategoryResponse,
  MonthlyReportResponse,
} from "../types/api";
import { formatMoney, formatMonth, getBarWidth } from "../utils/formatters";

const defaultFilters: ReportFilters = {};

export function ReportsPage() {
  const { token } = useAuth();

  const [categories, setCategories] = useState<CategoryResponse[]>([]);
  const [categoryReports, setCategoryReports] = useState<CategoryReportResponse[]>([]);
  const [monthlyReports, setMonthlyReports] = useState<MonthlyReportResponse[]>([]);

  const [filters, setFilters] = useState<ReportFilters>(defaultFilters);
  const [appliedFilters, setAppliedFilters] = useState<ReportFilters>(defaultFilters);

  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const maxCategoryAmount = useMemo(
    () => Math.max(0, ...categoryReports.map((item) => Number(item.total_amount))),
    [categoryReports],
  );

  const maxMonthlyAmount = useMemo(
    () => Math.max(0, ...monthlyReports.map((item) => Number(item.total_amount))),
    [monthlyReports],
  );

  useEffect(() => {
    if (!token) return;

    let ignore = false;

    async function loadReports() {
      setLoading(true);
      setError(null);

      try {
        const [categoryData, categorySummaryData, monthlySummaryData] =
          await Promise.all([
            listCategories(token),
            getCategorySummary(token, appliedFilters),
            getMonthlySummary(token, appliedFilters),
          ]);

        if (!ignore) {
          setCategories(categoryData);
          setCategoryReports(categorySummaryData);
          setMonthlyReports(monthlySummaryData);
        }
      } catch (err) {
        if (!ignore) {
          setError(getApiErrorMessage(err, "Failed to load reports"));
        }
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }

    void loadReports();

    return () => {
      ignore = true;
    };
  }, [token, appliedFilters]);

  function handleApplyFilters(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();
    setAppliedFilters({ ...filters });
  }

  function handleResetFilters() {
    setFilters(defaultFilters);
    setAppliedFilters(defaultFilters);
  }

  return (
    <section className="page-stack">
      <div className="page-card">
        <h1>Reports</h1>
        <p>Analyze spending by category and month.</p>

        {error && <div className="error-alert">{error}</div>}

        <form className="filters-grid reports-filter-grid" onSubmit={handleApplyFilters}>
          <div className="form-group">
            <label htmlFor="startDate">Start date</label>
            <input
              id="startDate"
              type="date"
              value={filters.start_date ?? ""}
              onChange={(event) =>
                setFilters({
                  ...filters,
                  start_date: event.target.value || undefined,
                })
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
                setFilters({
                  ...filters,
                  end_date: event.target.value || undefined,
                })
              }
            />
          </div>

          <div className="form-group">
            <label htmlFor="categoryId">Category</label>
            <select
              id="categoryId"
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

          <div className="filter-actions">
            <button className="primary-button">Apply filters</button>
            <button type="button" className="secondary-button" onClick={handleResetFilters}>
              Reset
            </button>
          </div>
        </form>
      </div>

      <div className="table-card">
        <h2>Category summary</h2>

        {loading ? (
          <p className="muted">Loading category report...</p>
        ) : categoryReports.length === 0 ? (
          <p className="muted">No category report data found.</p>
        ) : (
          <div className="report-list">
            {categoryReports.map((item) => (
              <div className="report-row" key={item.category_id}>
                <div className="report-row-header">
                  <div>
                    <strong>{item.category_name}</strong>
                    <span>{item.expense_count} expenses</span>
                  </div>

                  <strong>{formatMoney(item.total_amount)}</strong>
                </div>

                <div className="bar-track" aria-label={`${item.category_name} spending bar`}>
                  <div
                    className="bar-fill"
                    style={{
                      width: getBarWidth(item.total_amount, maxCategoryAmount),
                    }}
                  />
                </div>

                <p className="muted">
                  Average: {formatMoney(item.average_amount)}
                </p>
              </div>
            ))}
          </div>
        )}
      </div>

      <div className="table-card">
        <h2>Monthly summary</h2>

        {loading ? (
          <p className="muted">Loading monthly report...</p>
        ) : monthlyReports.length === 0 ? (
          <p className="muted">No monthly report data found.</p>
        ) : (
          <div className="report-list">
            {monthlyReports.map((item) => (
              <div className="report-row" key={item.month}>
                <div className="report-row-header">
                  <div>
                    <strong>{formatMonth(item.month)}</strong>
                    <span>{item.expense_count} expenses</span>
                  </div>

                  <strong>{formatMoney(item.total_amount)}</strong>
                </div>

                <div className="bar-track" aria-label={`${item.month} spending bar`}>
                  <div
                    className="bar-fill monthly"
                    style={{
                      width: getBarWidth(item.total_amount, maxMonthlyAmount),
                    }}
                  />
                </div>
              </div>
            ))}
          </div>
        )}
      </div>
    </section>
  );
}
```

## 6. Add CSS to `src/index.css`

Append:

```css
.quick-actions {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 16px;
  max-width: 900px;
}

.quick-action-card {
  padding: 20px;
  border-radius: 16px;
  background: #111827;
  color: white;
  font-weight: 800;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
}

.quick-action-card:hover {
  background: #2563eb;
}

.reports-filter-grid {
  grid-template-columns: repeat(4, minmax(0, 1fr));
}

.report-list {
  display: grid;
  gap: 18px;
}

.report-row {
  display: grid;
  gap: 10px;
  padding: 16px;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
}

.report-row-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.report-row-header div {
  display: grid;
  gap: 4px;
}

.report-row-header span {
  color: #6b7280;
  font-size: 14px;
}

.bar-track {
  width: 100%;
  height: 12px;
  overflow: hidden;
  border-radius: 999px;
  background: #e5e7eb;
}

.bar-fill {
  height: 100%;
  border-radius: 999px;
  background: #2563eb;
}

.bar-fill.monthly {
  background: #16a34a;
}

@media (max-width: 900px) {
  .quick-actions,
  .reports-filter-grid {
    grid-template-columns: 1fr;
  }

  .report-row-header {
    align-items: flex-start;
    flex-direction: column;
  }
}
```

## 7. Test in browser

Open:

```text
http://localhost:5173
```

Test:

```text
Login
Create categories if needed
Create expenses if needed
Open Dashboard
Check total amount, average, current month, top category
Open Reports
Check category summary
Check monthly summary
Apply start date and end date filters
Apply category filter
Reset filters
```

If reports show empty, create at least:

```text
Food    -> Breakfast 100
Food    -> Lunch 200
Travel  -> Cab 500
```

Expected:

```text
Dashboard total = 800
Top category = Travel
Category report shows Food and Travel
Monthly report shows current month
```

## 8. Build check

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run build
```

## 9. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 65 connect frontend reports dashboard"
```

Senior concept: reports should not be calculated in React from raw expenses. React should call reporting APIs. Backend/database handles aggregation; frontend handles presentation.

Sources checked: [React useEffect](https://react.dev/reference/react/useEffect), [React useMemo](https://react.dev/reference/react/useMemo), [MDN Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat/NumberFormat).
