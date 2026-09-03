Day 67 = Frontend automated tests.

Today we add tests for:

```text
API client
Login page
Register page
ProtectedRoute
```

## 1. Install test packages

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"

npm install -D vitest jsdom @testing-library/react @testing-library/dom @testing-library/jest-dom @testing-library/user-event
```

## 2. Update `package.json`

In `"scripts"`, add:

```json
"test": "vitest",
"test:run": "vitest run"
```

Your scripts should look similar to:

```json
"scripts": {
  "dev": "vite",
  "build": "tsc -b && vite build",
  "lint": "eslint .",
  "preview": "vite preview",
  "test": "vitest",
  "test:run": "vitest run"
}
```

## 3. Update `vite.config.ts`

Replace with:

```ts
import react from "@vitejs/plugin-react";
import { defineConfig } from "vitest/config";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: "./src/test/setup.ts",
    globals: true,
    css: true,
  },
});
```

## 4. Create `src/test/setup.ts`

```ts
import "@testing-library/jest-dom/vitest";
```

## 5. Create `src/api/client.test.ts`

```ts
import { afterEach, describe, expect, it, vi } from "vitest";

import { ApiError, apiRequest } from "./client";

afterEach(() => {
  vi.restoreAllMocks();
  vi.unstubAllGlobals();
});

describe("apiRequest", () => {
  it("sends JSON body and bearer token", async () => {
    const fetchMock = vi.fn().mockResolvedValue(
      new Response(JSON.stringify({ id: 1, name: "Food" }), {
        status: 200,
        headers: { "Content-Type": "application/json" },
      }),
    );

    vi.stubGlobal("fetch", fetchMock);

    const data = await apiRequest<{ id: number; name: string }>("/categories", {
      method: "POST",
      token: "test-token",
      body: { name: "Food" },
    });

    expect(data).toEqual({ id: 1, name: "Food" });

    const [, options] = fetchMock.mock.calls[0];
    const headers = options.headers as Headers;

    expect(headers.get("Authorization")).toBe("Bearer test-token");
    expect(headers.get("Content-Type")).toBe("application/json");
    expect(options.method).toBe("POST");
    expect(options.body).toBe(JSON.stringify({ name: "Food" }));
  });

  it("throws ApiError for failed responses", async () => {
    vi.stubGlobal(
      "fetch",
      vi.fn().mockResolvedValue(
        new Response(JSON.stringify({ detail: "Invalid request" }), {
          status: 400,
          headers: { "Content-Type": "application/json" },
        }),
      ),
    );

    await expect(apiRequest("/bad-request")).rejects.toMatchObject({
      status: 400,
      data: { detail: "Invalid request" },
    });

    await expect(apiRequest("/bad-request")).rejects.toBeInstanceOf(ApiError);
  });
});
```

## 6. Create `src/pages/LoginPage.test.tsx`

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { MemoryRouter, Route, Routes } from "react-router";
import { beforeEach, describe, expect, it, vi } from "vitest";

import { useAuth } from "../auth/AuthContext";
import { LoginPage } from "./LoginPage";

vi.mock("../auth/AuthContext", () => ({
  useAuth: vi.fn(),
}));

const mockedUseAuth = vi.mocked(useAuth);

function renderLoginPage() {
  render(
    <MemoryRouter initialEntries={["/login"]}>
      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/dashboard" element={<div>Dashboard Page</div>} />
      </Routes>
    </MemoryRouter>,
  );
}

beforeEach(() => {
  vi.clearAllMocks();
});

describe("LoginPage", () => {
  it("submits username and password", async () => {
    const loginMock = vi.fn().mockResolvedValue(undefined);

    mockedUseAuth.mockReturnValue({
      token: null,
      user: null,
      loading: false,
      isAuthenticated: false,
      register: vi.fn(),
      login: loginMock,
      logout: vi.fn(),
    });

    const user = userEvent.setup();

    renderLoginPage();

    await user.type(screen.getByLabelText(/username/i), "hari");
    await user.type(screen.getByLabelText(/password/i), "password123");
    await user.click(screen.getByRole("button", { name: /login/i }));

    await waitFor(() => {
      expect(loginMock).toHaveBeenCalledWith("hari", "password123");
    });
  });

  it("shows login error", async () => {
    const loginMock = vi.fn().mockRejectedValue(new Error("Invalid credentials"));

    mockedUseAuth.mockReturnValue({
      token: null,
      user: null,
      loading: false,
      isAuthenticated: false,
      register: vi.fn(),
      login: loginMock,
      logout: vi.fn(),
    });

    const user = userEvent.setup();

    renderLoginPage();

    await user.type(screen.getByLabelText(/username/i), "hari");
    await user.type(screen.getByLabelText(/password/i), "wrongpass");
    await user.click(screen.getByRole("button", { name: /login/i }));

    expect(await screen.findByText("Invalid credentials")).toBeInTheDocument();
  });
});
```

## 7. Create `src/pages/RegisterPage.test.tsx`

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { MemoryRouter, Route, Routes } from "react-router";
import { beforeEach, describe, expect, it, vi } from "vitest";

import { useAuth } from "../auth/AuthContext";
import { RegisterPage } from "./RegisterPage";

vi.mock("../auth/AuthContext", () => ({
  useAuth: vi.fn(),
}));

const mockedUseAuth = vi.mocked(useAuth);

function renderRegisterPage() {
  render(
    <MemoryRouter initialEntries={["/register"]}>
      <Routes>
        <Route path="/register" element={<RegisterPage />} />
        <Route path="/login" element={<div>Login Page</div>} />
      </Routes>
    </MemoryRouter>,
  );
}

beforeEach(() => {
  vi.clearAllMocks();
});

describe("RegisterPage", () => {
  it("registers a user and redirects to login", async () => {
    const registerMock = vi.fn().mockResolvedValue({
      id: 1,
      username: "hari",
      email: "hari@example.com",
      is_active: true,
    });

    mockedUseAuth.mockReturnValue({
      token: null,
      user: null,
      loading: false,
      isAuthenticated: false,
      register: registerMock,
      login: vi.fn(),
      logout: vi.fn(),
    });

    const user = userEvent.setup();

    renderRegisterPage();

    await user.type(screen.getByLabelText(/username/i), "hari");
    await user.type(screen.getByLabelText(/email/i), "hari@example.com");
    await user.type(screen.getByLabelText(/^password$/i), "password123");
    await user.type(screen.getByLabelText(/confirm password/i), "password123");
    await user.click(screen.getByRole("button", { name: /create account/i }));

    await waitFor(() => {
      expect(registerMock).toHaveBeenCalledWith({
        username: "hari",
        email: "hari@example.com",
        password: "password123",
      });
    });

    expect(await screen.findByText("Login Page")).toBeInTheDocument();
  });

  it("blocks mismatched passwords", async () => {
    const registerMock = vi.fn();

    mockedUseAuth.mockReturnValue({
      token: null,
      user: null,
      loading: false,
      isAuthenticated: false,
      register: registerMock,
      login: vi.fn(),
      logout: vi.fn(),
    });

    const user = userEvent.setup();

    renderRegisterPage();

    await user.type(screen.getByLabelText(/username/i), "hari");
    await user.type(screen.getByLabelText(/email/i), "hari@example.com");
    await user.type(screen.getByLabelText(/^password$/i), "password123");
    await user.type(screen.getByLabelText(/confirm password/i), "different123");
    await user.click(screen.getByRole("button", { name: /create account/i }));

    expect(await screen.findByText("Passwords do not match")).toBeInTheDocument();
    expect(registerMock).not.toHaveBeenCalled();
  });
});
```

## 8. Create `src/auth/ProtectedRoute.test.tsx`

```tsx
import { render, screen } from "@testing-library/react";
import { MemoryRouter, Route, Routes } from "react-router";
import { beforeEach, describe, expect, it, vi } from "vitest";

import { useAuth } from "./AuthContext";
import { ProtectedRoute } from "./ProtectedRoute";

vi.mock("./AuthContext", () => ({
  useAuth: vi.fn(),
}));

const mockedUseAuth = vi.mocked(useAuth);

function renderProtectedRoute() {
  render(
    <MemoryRouter initialEntries={["/expenses"]}>
      <Routes>
        <Route element={<ProtectedRoute />}>
          <Route path="/expenses" element={<div>Private Expenses Page</div>} />
        </Route>

        <Route path="/login" element={<div>Login Page</div>} />
      </Routes>
    </MemoryRouter>,
  );
}

beforeEach(() => {
  vi.clearAllMocks();
});

describe("ProtectedRoute", () => {
  it("shows loading state", () => {
    mockedUseAuth.mockReturnValue({
      token: null,
      user: null,
      loading: true,
      isAuthenticated: false,
      register: vi.fn(),
      login: vi.fn(),
      logout: vi.fn(),
    });

    renderProtectedRoute();

    expect(screen.getByText(/checking your login session/i)).toBeInTheDocument();
  });

  it("redirects unauthenticated users to login", () => {
    mockedUseAuth.mockReturnValue({
      token: null,
      user: null,
      loading: false,
      isAuthenticated: false,
      register: vi.fn(),
      login: vi.fn(),
      logout: vi.fn(),
    });

    renderProtectedRoute();

    expect(screen.getByText("Login Page")).toBeInTheDocument();
    expect(screen.queryByText("Private Expenses Page")).not.toBeInTheDocument();
  });

  it("allows authenticated users", () => {
    mockedUseAuth.mockReturnValue({
      token: "test-token",
      user: {
        id: 1,
        username: "hari",
        email: "hari@example.com",
        is_active: true,
      },
      loading: false,
      isAuthenticated: true,
      register: vi.fn(),
      login: vi.fn(),
      logout: vi.fn(),
    });

    renderProtectedRoute();

    expect(screen.getByText("Private Expenses Page")).toBeInTheDocument();
  });
});
```

## 9. Run tests

```powershell
npm run test:run
```

Expected:

```text
7 passed
```

## 10. Build check

```powershell
npm run build
```

## 11. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 67 add frontend tests"
```

Senior concept: frontend tests should verify user behavior, not internal state. Good tests ask: can the user login, see errors, get redirected, and trigger API calls correctly?

Sources checked: [Vitest getting started](https://vitest.dev/guide/), [React Testing Library API](https://testing-library.com/docs/react-testing-library/api/), [React Router MemoryRouter](https://reactrouter.com/api/declarative-routers/MemoryRouter), [Testing Library queries](https://testing-library.com/docs/queries/about/).
