Day 66 = Frontend CSV Exports.

Today we connect the frontend to the export APIs from Day 60.

```text
GET /exports/expenses.csv
GET /exports/category-summary.csv
GET /exports/monthly-summary.csv
```

Important: because these APIs need JWT auth, we download using `fetch()` with `Authorization: Bearer <token>`.

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

## 2. Create `src/api/exports.ts`

```ts
import { API_BASE_URL, ApiError } from "./client";

export type ExportType = "expenses" | "category-summary" | "monthly-summary";

export type ExportFilters = {
  q?: string;
  category_id?: string;
  start_date?: string;
  end_date?: string;
  min_amount?: string;
  max_amount?: string;
  sort_by?: "created_at" | "amount" | "title";
  sort_order?: "asc" | "desc";
};

const exportPaths: Record<ExportType, string> = {
  expenses: "/exports/expenses.csv",
  "category-summary": "/exports/category-summary.csv",
  "monthly-summary": "/exports/monthly-summary.csv",
};

function appendParam(params: URLSearchParams, key: string, value: string | undefined) {
  if (value && value.trim()) {
    params.set(key, value.trim());
  }
}

function buildExportQuery(type: ExportType, filters: ExportFilters) {
  const params = new URLSearchParams();

  appendParam(params, "start_date", filters.start_date);
  appendParam(params, "end_date", filters.end_date);

  if (type === "expenses") {
    appendParam(params, "q", filters.q);
    appendParam(params, "category_id", filters.category_id);
    appendParam(params, "min_amount", filters.min_amount);
    appendParam(params, "max_amount", filters.max_amount);
    appendParam(params, "sort_by", filters.sort_by);
    appendParam(params, "sort_order", filters.sort_order);
  }

  if (type === "category-summary") {
    appendParam(params, "category_id", filters.category_id);
  }

  const query = params.toString();

  return query ? `?${query}` : "";
}

function getDateStamp() {
  return new Date().toISOString().slice(0, 10);
}

function getFilename(type: ExportType) {
  return `${type}-${getDateStamp()}.csv`;
}

async function readErrorData(response: Response) {
  const contentType = response.headers.get("content-type");

  if (contentType?.includes("application/json")) {
    return response.json();
  }

  return response.text();
}

export async function downloadCsv(
  token: string,
  type: ExportType,
  filters: ExportFilters = {},
) {
  const response = await fetch(
    `${API_BASE_URL}${exportPaths[type]}${buildExportQuery(type, filters)}`,
    {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    },
  );

  if (!response.ok) {
    const data = await readErrorData(response);
    throw new ApiError(response.status, data);
  }

  const blob = await response.blob();
  const objectUrl = URL.createObjectURL(blob);

  const link = document.createElement("a");
  link.href = objectUrl;
  link.download = getFilename(type);

  document.body.appendChild(link);
  link.click();
  link.remove();

  window.setTimeout(() => {
    URL.revokeObjectURL(objectUrl);
  }, 1000);
}
```

## 3. Replace `src/pages/ExportsPage.tsx`

```tsx
import { useEffect, useState, type FormEvent } from "react";

import { listCategories } from "../api/categories";
import { getApiErrorMessage } from "../api/errors";
import {
  downloadCsv,
  type ExportFilters,
  type ExportType,
} from "../api/exports";
import { useAuth } from "../auth/AuthContext";
import type { CategoryResponse } from "../types/api";

const defaultFilters: ExportFilters = {
  sort_by: "created_at",
  sort_order: "desc",
};

type ExportCardProps = {
  title: string;
  description: string;
  note: string;
  buttonLabel: string;
  downloading: boolean;
  onDownload: () => void;
};

function ExportCard({
  title,
  description,
  note,
  buttonLabel,
  downloading,
  onDownload,
}: ExportCardProps) {
  return (
    <div className="export-card">
      <div>
        <h2>{title}</h2>
        <p>{description}</p>
        <span>{note}</span>
      </div>

      <button
        type="button"
        className="primary-button"
        disabled={downloading}
        onClick={onDownload}
      >
        {downloading ? "Downloading..." : buttonLabel}
      </button>
    </div>
  );
}

export function ExportsPage() {
  const { token } = useAuth();

  const [categories, setCategories] = useState<CategoryResponse[]>([]);
  const [filters, setFilters] = useState<ExportFilters>(defaultFilters);

  const [loadingCategories, setLoadingCategories] = useState(true);
  const [downloading, setDownloading] = useState<ExportType | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState<string | null>(null);

  useEffect(() => {
    if (!token) return;

    let ignore = false;

    async function loadCategories() {
      setLoadingCategories(true);
      setError(null);

      try {
        const data = await listCategories(token);

        if (!ignore) {
          setCategories(data);
        }
      } catch (err) {
        if (!ignore) {
          setError(getApiErrorMessage(err, "Failed to load categories"));
        }
      } finally {
        if (!ignore) {
          setLoadingCategories(false);
        }
      }
    }

    void loadCategories();

    return () => {
      ignore = true;
    };
  }, [token]);

  function handleFilterSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();
  }

  function handleResetFilters() {
    setFilters(defaultFilters);
    setError(null);
    setSuccess(null);
  }

  async function handleDownload(type: ExportType) {
    if (!token) return;

    setDownloading(type);
    setError(null);
    setSuccess(null);

    try {
      await downloadCsv(token, type, filters);
      setSuccess("CSV downloaded successfully");
    } catch (err) {
      setError(getApiErrorMessage(err, "Failed to download CSV"));
    } finally {
      setDownloading(null);
    }
  }

  return (
    <section className="page-stack">
      <div className="page-card">
        <h1>Exports</h1>
        <p>Download your expenses and reports as CSV files.</p>

        {error && <div className="error-alert">{error}</div>}
        {success && <div className="success-alert">{success}</div>}

        <form className="filters-grid exports-filter-grid" onSubmit={handleFilterSubmit}>
          <div className="form-group">
            <label htmlFor="q">Search title</label>
            <input
              id="q"
              value={filters.q ?? ""}
              onChange={(event) =>
                setFilters({ ...filters, q: event.target.value || undefined })
              }
              placeholder="Breakfast"
            />
            <small>Expenses export only</small>
          </div>

          <div className="form-group">
            <label htmlFor="categoryId">Category</label>
            <select
              id="categoryId"
              value={filters.category_id ?? ""}
              disabled={loadingCategories}
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
            <small>Expenses and category-summary exports</small>
          </div>

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
            <label htmlFor="minAmount">Min amount</label>
            <input
              id="minAmount"
              type="number"
              min="0"
              step="0.01"
              value={filters.min_amount ?? ""}
              onChange={(event) =>
                setFilters({
                  ...filters,
                  min_amount: event.target.value || undefined,
                })
              }
            />
            <small>Expenses export only</small>
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
                setFilters({
                  ...filters,
                  max_amount: event.target.value || undefined,
                })
              }
            />
            <small>Expenses export only</small>
          </div>

          <div className="form-group">
            <label htmlFor="sortBy">Sort by</label>
            <select
              id="sortBy"
              value={filters.sort_by ?? "created_at"}
              onChange={(event) =>
                setFilters({
                  ...filters,
                  sort_by: event.target.value as ExportFilters["sort_by"],
                })
              }
            >
              <option value="created_at">Created date</option>
              <option value="amount">Amount</option>
              <option value="title">Title</option>
            </select>
            <small>Expenses export only</small>
          </div>

          <div className="form-group">
            <label htmlFor="sortOrder">Sort order</label>
            <select
              id="sortOrder"
              value={filters.sort_order ?? "desc"}
              onChange={(event) =>
                setFilters({
                  ...filters,
                  sort_order: event.target.value as ExportFilters["sort_order"],
                })
              }
            >
              <option value="desc">Descending</option>
              <option value="asc">Ascending</option>
            </select>
            <small>Expenses export only</small>
          </div>

          <div className="filter-actions">
            <button type="button" className="secondary-button" onClick={handleResetFilters}>
              Reset filters
            </button>
          </div>
        </form>
      </div>

      <div className="export-grid">
        <ExportCard
          title="Expenses CSV"
          description="Detailed export of individual expense rows."
          note="Uses all filters: search, category, date, amount, and sorting."
          buttonLabel="Download expenses"
          downloading={downloading === "expenses"}
          onDownload={() => void handleDownload("expenses")}
        />

        <ExportCard
          title="Category Summary CSV"
          description="Aggregated spending grouped by category."
          note="Uses date and category filters."
          buttonLabel="Download category summary"
          downloading={downloading === "category-summary"}
          onDownload={() => void handleDownload("category-summary")}
        />

        <ExportCard
          title="Monthly Summary CSV"
          description="Aggregated spending grouped by month."
          note="Uses date filters only."
          buttonLabel="Download monthly summary"
          downloading={downloading === "monthly-summary"}
          onDownload={() => void handleDownload("monthly-summary")}
        />
      </div>
    </section>
  );
}
```

## 4. Add CSS to `src/index.css`

Append:

```css
.exports-filter-grid {
  grid-template-columns: repeat(4, minmax(0, 1fr));
}

.form-group small {
  color: #6b7280;
  font-size: 12px;
}

.export-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 18px;
  max-width: 1100px;
}

.export-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 24px;
  min-height: 260px;
  padding: 24px;
  border-radius: 16px;
  background: white;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
}

.export-card h2 {
  margin: 0 0 10px;
}

.export-card p {
  margin: 0 0 14px;
  color: #4b5563;
}

.export-card span {
  display: block;
  color: #6b7280;
  font-size: 14px;
}

@media (max-width: 900px) {
  .exports-filter-grid,
  .export-grid {
    grid-template-columns: 1fr;
  }
}
```

## 5. Test in browser

Open:

```text
http://localhost:5173/exports
```

Test:

```text
Login
Go to Exports
Download Expenses CSV
Download Category Summary CSV
Download Monthly Summary CSV
Apply date filter
Download again
Apply category filter
Download Expenses CSV
Download Category Summary CSV
```

Expected:

```text
CSV files download
Only logged-in user's data appears
Filters affect exported rows
No token appears in URL
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
git commit -m "Day 66 connect frontend CSV exports"
```

Senior concept: authenticated downloads should not expose tokens in URLs. Use `fetch()` with an `Authorization` header, convert the response to a `Blob`, create a temporary object URL, click a hidden download link, then revoke the object URL.

Sources checked: [MDN Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch), [MDN Response.blob](https://developer.mozilla.org/en-US/docs/Web/API/Response/blob), [MDN anchor download](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/a), [FastAPI custom responses](https://fastapi.tiangolo.com/advanced/custom-response/).
