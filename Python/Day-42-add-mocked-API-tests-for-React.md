Regular `Day 42`: mock API testing in React.

Today you test React pages without running FastAPI. This is important because frontend tests should be fast and reliable. We’ll mock `fetch()` using Vitest.

Sources checked: Vitest supports global mocking with `vi.stubGlobal`; Testing Library recommends `userEvent.setup()` for realistic interactions; TanStack Query tests should use a fresh `QueryClient` with retries disabled. See [Vitest mocking](https://github.com/vitest-dev/vitest/blob/main/docs/guide/mocking.md), [Testing Library user-event](https://testing-library.com/docs/user-event/setup/), and [TanStack Query testing](https://tanstack.com/query/latest/docs/framework/react/guides/testing).

**Day 42 Goal**

- Mock API responses
- Test page loading state/data
- Test form submission
- Test failed API response
- Test TanStack Query pages without backend
- Understand frontend integration tests

Start from Day 41:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-41-react-testing .\day-42-react-api-mock-tests
cd day-42-react-api-mock-tests
code .
```

Update `src/test/setup.js`:

```javascript
import "@testing-library/jest-dom/vitest";
import { afterEach, vi } from "vitest";

afterEach(() => {
  localStorage.clear();
  vi.unstubAllGlobals();
});
```

Create `src/api/auth.api.test.js`:

```javascript
import { describe, expect, it, vi } from "vitest";
import { loginUser, registerUser } from "./auth";

describe("auth API", () => {
  it("registers a user", async () => {
    vi.stubGlobal(
      "fetch",
      vi.fn().mockResolvedValue({
        ok: true,
        json: async () => ({ id: 1, username: "hari" }),
      })
    );

    const user = await registerUser({
      username: "hari",
      email: "hari@example.com",
      password: "secret123",
    });

    expect(user.username).toBe("hari");
    expect(fetch).toHaveBeenCalledWith(
      "http://127.0.0.1:8000/auth/register",
      expect.objectContaining({ method: "POST" })
    );
  });

  it("logs in and returns token", async () => {
    vi.stubGlobal(
      "fetch",
      vi.fn().mockResolvedValue({
        ok: true,
        json: async () => ({ access_token: "abc123", token_type: "bearer" }),
      })
    );

    const result = await loginUser("hari", "secret123");

    expect(result.access_token).toBe("abc123");
  });
});
```

Create `src/pages/CategoriesPage.test.jsx`:

```jsx
import { screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, expect, it, vi } from "vitest";
import CategoriesPage from "./CategoriesPage";
import { renderWithProviders } from "../test/test-utils";

describe("CategoriesPage", () => {
  it("loads categories from API", async () => {
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

    renderWithProviders(<CategoriesPage />);

    expect(await screen.findByText("Food")).toBeInTheDocument();
    expect(await screen.findByText("Travel")).toBeInTheDocument();
  });

  it("creates a category", async () => {
    const user = userEvent.setup();

    vi.stubGlobal(
      "fetch",
      vi
        .fn()
        .mockResolvedValueOnce({
          ok: true,
          json: async () => [],
        })
        .mockResolvedValueOnce({
          ok: true,
          json: async () => ({ id: 1, name: "Study" }),
        })
        .mockResolvedValueOnce({
          ok: true,
          json: async () => [{ id: 1, name: "Study" }],
        })
    );

    renderWithProviders(<CategoriesPage />);

    await user.type(screen.getByPlaceholderText("Category name"), "Study");
    await user.click(screen.getByRole("button", { name: /add category/i }));

    expect(await screen.findByText("Study")).toBeInTheDocument();
  });
});
```

Run tests:

```powershell
npm run test:run
```

Expected now: previous Day 41 tests plus new tests should pass.

**Challenge**

Add one test for failed category loading:

- Mock `fetch()` with `ok: false`
- Render `CategoriesPage`
- Confirm error message appears

Commit:

```powershell
git add .
git commit -m "Day 42 add mocked API tests for React"
```

Day 42 is complete when you can explain: mocked API tests prove the React UI behaves correctly without needing FastAPI or PostgreSQL running.
