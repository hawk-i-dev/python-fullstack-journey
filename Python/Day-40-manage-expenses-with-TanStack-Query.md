`Day 40`: finish TanStack Query for protected expenses.

Day 39 added TanStack Query for categories/reports. Today you migrate the main expenses page: list, filters, pagination, create, update, delete.

**Day 40 Goal**

- Use `useQuery()` for filtered expenses
- Include filters in the query key
- Use `useMutation()` for create/update/delete
- Invalidate affected queries after changes
- Remove manual `loadData()` calls
- Keep JWT auth and user-owned backend filtering

Start from Day 39:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-39-react-query .\day-40-react-query-expenses
cd day-40-react-query-expenses
code .
```

Create `src/hooks/useExpensesQuery.js`:

```javascript
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import {
  createExpense,
  deleteExpense,
  getExpenses,
  updateExpense,
} from "../api/expenses";

export function useExpensesQuery(filters) {
  return useQuery({
    queryKey: ["expenses", filters],
    queryFn: () => getExpenses(filters),
  });
}

export function useCreateExpenseMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createExpense,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["expenses"] });
      queryClient.invalidateQueries({ queryKey: ["expense-summary"] });
      queryClient.invalidateQueries({ queryKey: ["category-report"] });
    },
  });
}

export function useUpdateExpenseMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ expenseId, expense }) => updateExpense(expenseId, expense),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["expenses"] });
      queryClient.invalidateQueries({ queryKey: ["expense-summary"] });
      queryClient.invalidateQueries({ queryKey: ["category-report"] });
    },
  });
}

export function useDeleteExpenseMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deleteExpense,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["expenses"] });
      queryClient.invalidateQueries({ queryKey: ["expense-summary"] });
      queryClient.invalidateQueries({ queryKey: ["category-report"] });
    },
  });
}
```

In `src/pages/ExpensesPage.jsx`, replace manual loading with these hooks:

```jsx
const skip = (filters.page - 1) * filters.limit;

const queryFilters = {
  search: filters.search,
  categoryId: filters.categoryId,
  sortBy: filters.sortBy,
  sortOrder: filters.sortOrder,
  skip,
  limit: filters.limit,
};

const {
  data: expenses = [],
  isLoading,
  error,
} = useExpensesQuery(queryFilters);

const { data: categories = [] } = useCategoriesQuery();

const createMutation = useCreateExpenseMutation();
const updateMutation = useUpdateExpenseMutation();
const deleteMutation = useDeleteExpenseMutation();
```

Update submit logic:

```jsx
async function handleSubmit(data) {
  const expense = {
    title: data.title.trim(),
    amount: Number(data.amount),
    category_id: Number(data.category_id),
  };

  if (editingExpense) {
    await updateMutation.mutateAsync({
      expenseId: editingExpense.id,
      expense,
    });
    setEditingExpense(null);
  } else {
    await createMutation.mutateAsync(expense);
  }

  reset();
}
```

Update delete:

```jsx
async function handleDelete(expenseId) {
  await deleteMutation.mutateAsync(expenseId);
}
```

Show mutation errors:

```jsx
{error && <p className="error">{error.message}</p>}
{createMutation.error && <p className="error">{createMutation.error.message}</p>}
{updateMutation.error && <p className="error">{updateMutation.error.message}</p>}
{deleteMutation.error && <p className="error">{deleteMutation.error.message}</p>}
```

Disable buttons while saving:

```jsx
<button
  disabled={createMutation.isPending || updateMutation.isPending}
  type="submit"
>
  {editingExpense ? "Update Expense" : "Add Expense"}
</button>
```

Run backend:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-21-user-owned-expenses"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Run React:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-40-react-query-expenses"
npm run dev
```

Test:

- Login
- Add expense
- Edit expense
- Delete expense
- Search/filter/sort still work
- Reports update after expense changes
- Pagination still works

Commit:

```powershell
git add .
git commit -m "Day 40 manage expenses with TanStack Query"
```

Day 40 is complete when you can explain: filter values belong in the query key because changing filters should produce a different cached API result.
