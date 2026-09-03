Day 68 = Playwright E2E tests.

Today we test the full app in a real browser:

```text
React UI → FastAPI API → PostgreSQL DB
```

## 1. Install Playwright

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"

npm install -D @playwright/test
npx playwright install chromium
```

## 2. Update `frontend/package.json`

Add scripts:

```json
"test:e2e": "playwright test",
"test:e2e:headed": "playwright test --headed",
"test:e2e:ui": "playwright test --ui"
```

Example:

```json
"scripts": {
  "dev": "vite",
  "build": "tsc -b && vite build",
  "lint": "eslint .",
  "preview": "vite preview",
  "test": "vitest",
  "test:run": "vitest run",
  "test:e2e": "playwright test",
  "test:e2e:headed": "playwright test --headed",
  "test:e2e:ui": "playwright test --ui"
}
```

## 3. Create `frontend/playwright.config.ts`

```ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  timeout: 30_000,
  fullyParallel: false,
  reporter: [["list"], ["html"]],
  use: {
    baseURL: "http://127.0.0.1:5173",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  webServer: {
    command: "npm run dev -- --host 127.0.0.1",
    url: "http://127.0.0.1:5173",
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
  ],
});
```

## 4. Update `frontend/.gitignore`

Add:

```gitignore
playwright-report/
test-results/
```

## 5. Create `frontend/e2e/expense-manager.spec.ts`

```ts
import { expect, test } from "@playwright/test";

function uniqueValue(prefix: string) {
  return `${prefix}_${Date.now()}_${Math.floor(Math.random() * 10000)}`;
}

test("redirects logged out users away from private pages", async ({ page }) => {
  await page.goto("/expenses");

  await expect(page).toHaveURL(/\/login/);
  await expect(page.getByRole("heading", { name: "Login" })).toBeVisible();
});

test("user can register, login, create category, create expense, view reports, and export CSV", async ({
  page,
}) => {
  const username = uniqueValue("e2e_user");
  const email = `${username}@example.com`;
  const password = "password123";
  const categoryName = uniqueValue("Food");
  const expenseTitle = uniqueValue("Breakfast");

  await page.goto("/register");

  await page.getByLabel("Username").fill(username);
  await page.getByLabel("Email").fill(email);
  await page.getByLabel(/^Password$/).fill(password);
  await page.getByLabel("Confirm password").fill(password);
  await page.getByRole("button", { name: "Create account" }).click();

  await expect(page.getByRole("heading", { name: "Login" })).toBeVisible();

  await page.getByLabel("Username").fill(username);
  await page.getByLabel("Password").fill(password);
  await page.getByRole("button", { name: "Login" }).click();

  await expect(page.getByRole("heading", { name: "Dashboard" })).toBeVisible();
  await expect(page.getByText(username)).toBeVisible();

  await page.getByRole("link", { name: "Categories" }).click();

  await page.getByLabel("Category name").fill(categoryName);
  await page.getByRole("button", { name: "Add category" }).click();

  await expect(page.getByText(categoryName)).toBeVisible();

  await page.getByRole("link", { name: "Expenses" }).click();

  await page.getByLabel("Title").fill(expenseTitle);
  await page.getByLabel("Amount").fill("500");
  await page.locator("#category").selectOption({ label: categoryName });
  await page.getByRole("button", { name: "Add expense" }).click();

  await expect(page.getByText(expenseTitle)).toBeVisible();
  await expect(page.getByText(categoryName)).toBeVisible();

  await page.getByRole("link", { name: "Reports" }).click();

  await expect(page.getByRole("heading", { name: "Reports" })).toBeVisible();
  await expect(page.getByText(categoryName).first()).toBeVisible();

  await page.getByRole("link", { name: "Exports" }).click();

  const downloadPromise = page.waitForEvent("download");
  await page.getByRole("button", { name: "Download expenses" }).click();

  const download = await downloadPromise;

  expect(download.suggestedFilename()).toMatch(/^expenses-\d{4}-\d{2}-\d{2}\.csv$/);
});
```

## 6. Run backend manually

Keep backend running in Terminal 1:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\backend"
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```

Playwright will start the frontend automatically.

## 7. Run E2E tests

Terminal 2:

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas\frontend"
npm run test:e2e
```

Expected:

```text
2 passed
```

To watch browser actions:

```powershell
npm run test:e2e:headed
```

For Playwright UI mode:

```powershell
npm run test:e2e:ui
```

## 8. If test fails

Open the HTML report:

```powershell
npx playwright show-report
```

Common causes:

```text
Backend not running on 127.0.0.1:8000
Frontend .env missing VITE_API_BASE_URL=http://127.0.0.1:8000
Database migrations not applied
Category/expense backend route has error
CORS not configured
```

## 9. Run all frontend checks

```powershell
npm run test:run
npm run test:e2e
npm run build
```

## 10. Commit

```powershell
cd "$env:USERPROFILE\Documents\expense-manager-saas"

git status
git add .
git commit -m "Day 68 add Playwright end to end tests"
```

Senior concept: unit tests check pieces; E2E tests check the real user journey. A small number of strong E2E tests is better than many fragile UI tests.

Sources checked: [Playwright installation](https://playwright.dev/docs/intro), [Playwright webServer](https://playwright.dev/docs/test-webserver), [Playwright locators](https://playwright.dev/docs/locators).
