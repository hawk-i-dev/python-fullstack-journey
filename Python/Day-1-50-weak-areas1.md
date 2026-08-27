Let’s cover the 4 weak areas properly.

**1. `useEffect()`**

`useState()` stores data.  
`useEffect()` runs side-effect code, usually API calls, after the component renders.

Example:

```jsx
const [expenses, setExpenses] = useState([]);

useEffect(() => {
  async function loadExpenses() {
    const response = await fetch("http://127.0.0.1:8000/expenses");
    const data = await response.json();
    setExpenses(data);
  }

  loadExpenses();
}, []);
```

Meaning of `[]`:

```jsx
useEffect(() => {
  // runs once when page opens
}, []);
```

With filters:

```jsx
useEffect(() => {
  loadExpenses();
}, [filters]);
```

This means: whenever `filters` changes, call API again.

Simple rule:

```text
useState = store value
useEffect = run API call / side effect
```

**2. `invalidateQueries()`**

TanStack Query caches API data.

Suppose this query loads expenses:

```jsx
useQuery({
  queryKey: ["expenses"],
  queryFn: getExpenses,
});
```

Then user adds a new expense:

```jsx
useMutation({
  mutationFn: createExpense,
});
```

Problem: React Query still has old cached expenses.

Fix:

```jsx
const queryClient = useQueryClient();

useMutation({
  mutationFn: createExpense,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["expenses"] });
  },
});
```

Meaning:

```text
invalidateQueries = this cached data is old, fetch it again
```

For expense app, after add/update/delete, invalidate:

```javascript
["expenses"]
["expense-summary"]
["category-report"]
```

Because all three may change.

**3. Mocked Frontend Tests**

Frontend tests should not depend on FastAPI/PostgreSQL.

Bad testing dependency:

```text
React test -> needs FastAPI running -> needs PostgreSQL running
```

Better:

```text
React test -> fake API response -> test UI behavior
```

Example:

```javascript
vi.stubGlobal(
  "fetch",
  vi.fn().mockResolvedValue({
    ok: true,
    json: async () => [
      { id: 1, name: "Food" },
      { id: 2, name: "Travel" },
    ],
  })
);
```

Then test:

```jsx
renderWithProviders(<CategoriesPage />);

expect(await screen.findByText("Food")).toBeInTheDocument();
```

What this proves:

```text
If API returns Food and Travel, React shows Food and Travel.
```

It does not test database. Backend tests handle that separately.

**4. Docker Compose**

Docker runs one container.

Docker Compose runs multiple containers together.

Your app needs:

```text
frontend container
backend/api container
database container
```

So use Compose:

```yaml
services:
  db:
    image: postgres:17

  api:
    build: ./backend
    depends_on:
      - db

  frontend:
    build: ./frontend
    depends_on:
      - api
```

Inside Compose, backend connects to database using service name:

```env
DB_HOST=db
```

Not:

```env
DB_HOST=localhost
```

Because inside the API container, `localhost` means the API container itself, not the database container.

Simple rule:

```text
Docker = run one packaged app
Docker Compose = run full system with multiple services
```

**Quick Re-Test**

Answer these 4:

1. Which hook should load API data when a page opens?  
A. `useState`  
B. `useEffect`

2. After `createExpense()` succeeds, what should TanStack Query do?  
A. `invalidateQueries()`  
B. Delete the database

3. Should frontend tests require FastAPI/PostgreSQL running?  
A. Yes  
B. No, mock API calls

4. What runs frontend, backend, and database together?  
A. Docker Compose  
B. Pydantic
