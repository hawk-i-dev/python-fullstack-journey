Regular `Day 41`: frontend testing with Vitest + React Testing Library.

Today you start testing the React app. After Day 40, your app has auth, protected routes, forms, API calls, and TanStack Query. Now you need automated tests so changes do not silently break login/navigation behavior.

Sources checked: Vitest supports React/Vite testing with `jsdom`; TanStack Query recommends creating a fresh `QueryClient` for each test and disabling retries. See [Vitest features](https://vitest.dev/guide/features) and [TanStack Query testing](https://tanstack.com/query/latest/docs/framework/react/guides/testing).

**Day 41 Goal**

- Install frontend test tools
- Configure Vitest
- Test localStorage auth helpers
- Test protected route redirect
- Create reusable test utilities
- Learn `render`, `screen`, `expect`

Start from Day 40:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-40-react-query-expenses .\day-41-react-testing
cd day-41-react-testing
npm install
npm install -D vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
code .
```

Update `package.json` scripts:

```json
{
  "scripts": {
    "dev": "vite",
    "test": "vitest",
    "test:run": "vitest run",
    "build": "vite build"
  }
}
```

Update `vite.config.js`:

```javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: "./src/test/setup.js",
  },
});
```

Create:

```text
src/test/setup.js
src/test/test-utils.jsx
```

`src/test/setup.js`:

```javascript
import "@testing-library/jest-dom/vitest";
```

`src/test/test-utils.jsx`:

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { render } from "@testing-library/react";
import { MemoryRouter } from "react-router";

export function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });
}

export function renderWithProviders(ui, { route = "/" } = {}) {
  const queryClient = createTestQueryClient();

  return render(
    <QueryClientProvider client={queryClient}>
      <MemoryRouter initialEntries={[route]}>{ui}</MemoryRouter>
    </QueryClientProvider>
  );
}
```

Create `src/api/auth.test.js`:

```javascript
import { describe, expect, it, beforeEach } from "vitest";
import { getToken, logout, saveToken } from "./auth";

describe("auth token helpers", () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it("saves and reads token", () => {
    saveToken("test-token");

    expect(getToken()).toBe("test-token");
  });

  it("removes token on logout", () => {
    saveToken("test-token");
    logout();

    expect(getToken()).toBeNull();
  });
});
```

Create `src/components/ProtectedRoute.test.jsx`:

```jsx
import { Route, Routes } from "react-router";
import { describe, expect, it, beforeEach } from "vitest";
import { screen } from "@testing-library/react";
import ProtectedRoute from "./ProtectedRoute";
import { saveToken } from "../api/auth";
import { renderWithProviders } from "../test/test-utils";

function TestRoutes() {
  return (
    <Routes>
      <Route path="/login" element={<p>Login page</p>} />

      <Route element={<ProtectedRoute />}>
        <Route path="/profile" element={<p>Profile page</p>} />
      </Route>
    </Routes>
  );
}

describe("ProtectedRoute", () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it("redirects to login when token is missing", () => {
    renderWithProviders(<TestRoutes />, { route: "/profile" });

    expect(screen.getByText("Login page")).toBeInTheDocument();
  });

  it("shows protected page when token exists", () => {
    saveToken("valid-token");

    renderWithProviders(<TestRoutes />, { route: "/profile" });

    expect(screen.getByText("Profile page")).toBeInTheDocument();
  });
});
```

Run tests:

```powershell
npm run test:run
```

Expected result:

```text
3 passed
```

Run dev app also:

```powershell
npm run dev
```

**Challenge**

Add one test for `LoginPage`:

- Render login page
- Confirm username input exists
- Confirm password input exists
- Confirm login button exists

Commit:

```powershell
git add .
git commit -m "Day 41 add frontend tests with Vitest"
```

Day 41 is complete when you can explain: Vitest runs the tests, React Testing Library checks what the user sees, and test utilities wrap components with the same providers used by the real app.
