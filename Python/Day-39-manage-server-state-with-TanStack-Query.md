`Day 39`: TanStack Query for React server state.

Today you replace manual `useEffect + loading + refetch` patterns with TanStack Query. This is how professional React apps usually manage API data, caching, refetching, and mutations. Source checked: [TanStack Query install](https://tanstack.com/query/latest/docs/framework/react/installation), [query keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys).

**Day 39 Goal**

- Add TanStack Query
- Cache API responses
- Use `useQuery()` for loading data
- Use `useMutation()` for create/update/delete
- Invalidate cached data after changes
- Reduce manual `loadData()` calls

Start from Day 38:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-38-react-form-validation .\day-39-react-query
cd day-39-react-query
npm install @tanstack/react-query
code .
```

Update `src/main.jsx`:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import App from "./App.jsx";

const queryClient = new QueryClient();

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </QueryClientProvider>
  </StrictMode>
);
```

Create `src/hooks/useCategoriesQuery.js`:

```javascript
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { createCategory, getCategories } from "../api/expenses";

export function useCategoriesQuery() {
  return useQuery({
    queryKey: ["categories"],
    queryFn: getCategories,
  });
}

export function useCreateCategoryMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createCategory,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["categories"] });
    },
  });
}
```

Update `src/pages/CategoriesPage.jsx` to use Query:

```jsx
import { useState } from "react";
import {
  useCategoriesQuery,
  useCreateCategoryMutation,
} from "../hooks/useCategoriesQuery";

function CategoriesPage() {
  const [name, setName] = useState("");
  const { data: categories = [], isLoading, error } = useCategoriesQuery();
  const createCategoryMutation = useCreateCategoryMutation();

  async function handleSubmit(event) {
    event.preventDefault();

    const category = { name: name.trim() };

    if (!category.name) {
      return;
    }

    await createCategoryMutation.mutateAsync(category);
    setName("");
  }

  if (isLoading) {
    return <p>Loading categories...</p>;
  }

  return (
    <>
      <h1>Categories</h1>

      {error && <p className="error">{error.message}</p>}
      {createCategoryMutation.error && (
        <p className="error">{createCategoryMutation.error.message}</p>
      )}

      <form onSubmit={handleSubmit}>
        <input
          value={name}
          onChange={(event) => setName(event.target.value)}
          placeholder="Category name"
        />

        <button disabled={createCategoryMutation.isPending} type="submit">
          {createCategoryMutation.isPending ? "Saving..." : "Add Category"}
        </button>
      </form>

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

Create `src/hooks/useReportsQuery.js`:

```javascript
import { useQuery } from "@tanstack/react-query";
import { getCategoryReport, getExpenseSummary } from "../api/expenses";

export function useExpenseSummaryQuery() {
  return useQuery({
    queryKey: ["expense-summary"],
    queryFn: getExpenseSummary,
  });
}

export function useCategoryReportQuery() {
  return useQuery({
    queryKey: ["category-report"],
    queryFn: getCategoryReport,
  });
}
```

In `ReportsPage.jsx`, replace the old report loading hook with:

```jsx
const {
  data: summary = { total: 0, count: 0 },
  isLoading: summaryLoading,
  error: summaryError,
} = useExpenseSummaryQuery();

const {
  data: categoryReport = [],
  isLoading: reportLoading,
  error: reportError,
} = useCategoryReportQuery();

if (summaryLoading || reportLoading) {
  return <p>Loading reports...</p>;
}
```

Then use:

```jsx
{summaryError && <p className="error">{summaryError.message}</p>}
{reportError && <p className="error">{reportError.message}</p>}
```

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run React:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-39-react-query"
npm run dev
```

Test:

- Login
- Open `/categories`
- Add category
- It should appear without manual refresh
- Open `/reports`
- Charts should still load
- Refresh browser and confirm data still loads

Commit:

```powershell
git add .
git commit -m "Day 39 manage server state with TanStack Query"
```

Day 39 is complete when you can explain: `useQuery()` reads and caches server data, `useMutation()` changes server data, and `invalidateQueries()` tells React to refetch stale data after a change.
