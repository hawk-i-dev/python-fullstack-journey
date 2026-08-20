Regular `Day 43`: end-to-end testing with Playwright.

Today you test the app in a real browser. Unlike Day 41/42 component tests, Playwright opens the actual React app and checks user flows. We will mock API responses so FastAPI/PostgreSQL do not need to run.

Sources checked: Playwright official docs recommend `npm init playwright@latest`, support `webServer` for starting Vite during tests, and `page.route()` for mocking API requests. See [Playwright install](https://playwright.dev/docs/intro), [mock APIs](https://playwright.dev/docs/mock), and [webServer config](https://playwright.dev/docs/api/class-testconfig).

**Day 43 Goal**

- Install Playwright
- Start Vite automatically during tests
- Test login flow
- Test protected expenses page
- Mock backend API calls
- Understand E2E testing

Start from Day 42:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-42-react-api-mock-tests .\day-43-playwright-e2e
cd day-43-playwright-e2e
npm install
npm init playwright@latest
code .
```

When prompted, choose:

```text
TypeScript or JavaScript: JavaScript
Tests folder: e2e
GitHub Actions workflow: No
Install Playwright browsers: Yes
```

Update `playwright.config.js`:

```javascript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  retries: 0,
  reporter: "html",
  use: {
    baseURL: "http://localhost:5173",
    trace: "on-first-retry",
  },
  webServer: {
    command: "npm run dev",
    url: "http://localhost:5173",
    reuseExistingServer: true,
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
  ],
});
```

Create `e2e/auth.spec.js`:

```javascript
import { expect, test } from "@playwright/test";

test("user can login and view profile", async ({ page }) => {
  await page.route("**/auth/login", async (route) => {
    await route.fulfill({
      status: 200,
      contentType: "application/json",
      body: JSON.stringify({
        access_token: "fake-token",
        token_type: "bearer",
      }),
    });
  });

  await page.route("**/auth/me", async (route) => {
    await route.fulfill({
      status: 200,
      contentType: "application/json",
      body: JSON.stringify({
        id: 1,
        username: "hari",
        email: "hari@example.com",
        is_active: true,
      }),
    });
  });

  await page.goto("/login");

  await page.getByPlaceholder("Username").fill("hari");
  await page.getByPlaceholder("Password").fill("secret123");
  await page.getByRole("button", { name: "Login" }).click();

  await expect(page.getByText("Username: hari")).toBeVisible();
  await expect(page.getByText("Email: hari@example.com")).toBeVisible();
});
```

Create `e2e/expenses.spec.js`:

```javascript
import { expect, test } from "@playwright/test";

test("logged in user can view protected expenses", async ({ page }) => {
  await page.addInitScript(() => {
    localStorage.setItem("access_token", "fake-token");
  });

  await page.route("**/categories", async (route) => {
    await route.fulfill({
      status: 200,
      contentType: "application/json",
      body: JSON.stringify([
        { id: 1, name: "Food" },
        { id: 2, name: "Travel" },
      ]),
    });
  });

  await page.route("**/expenses?**", async (route) => {
    await route.fulfill({
      status: 200,
      contentType: "application/json",
      body: JSON.stringify([
        {
          id: 1,
          title: "Tea",
          amount: 20,
          category: { id: 1, name: "Food" },
        },
        {
          id: 2,
          title: "Bus",
          amount: 35,
          category: { id: 2, name: "Travel" },
        },
      ]),
    });
  });

  await page.goto("/expenses");

  await expect(page.getByText("Tea")).toBeVisible();
  await expect(page.getByText("Bus")).toBeVisible();
  await expect(page.getByText("Food")).toBeVisible();
  await expect(page.getByText("Travel")).toBeVisible();
});
```

Run tests:

```powershell
npx playwright test
```

Open report:

```powershell
npx playwright show-report
```

Run headed mode to watch browser actions:

```powershell
npx playwright test --headed
```

**Challenge**

Add one E2E test:

- Open `/expenses` without token
- Confirm it redirects to `/login`

Hint:

```javascript
await page.goto("/expenses");
await expect(page).toHaveURL(/login/);
```

Commit:

```powershell
git add .
git commit -m "Day 43 add Playwright end-to-end tests"
```

Day 43 is complete when you can explain: Vitest tests components in a simulated DOM, while Playwright tests real browser behavior across pages, navigation, forms, and API calls.
