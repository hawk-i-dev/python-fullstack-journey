Regular `Day 44`: GitHub Actions CI for React.

Today you make GitHub automatically run your frontend checks: unit tests, production build, and Playwright E2E tests. Official Playwright CI docs currently recommend `npm ci`, `npx playwright install --with-deps`, and `npx playwright test`. They also recommend `workers: 1` in CI for stability. Sources: [Playwright CI](https://playwright.dev/docs/ci), [Playwright webServer](https://playwright.dev/docs/test-webserver), [Vite build](https://vite.dev/guide/build.html).

**Day 44 Goal**

- Add GitHub Actions workflow
- Run Vitest in CI
- Run Vite production build in CI
- Run Playwright in CI
- Upload Playwright report artifact
- Understand CI/CD basics

Start from Day 43:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
Copy-Item -Recurse .\day-43-playwright-e2e .\day-44-github-actions-ci
cd day-44-github-actions-ci
code .
```

Update `playwright.config.js`:

```javascript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  workers: process.env.CI ? 1 : undefined,
  retries: process.env.CI ? 1 : 0,
  reporter: "html",
  use: {
    baseURL: "http://127.0.0.1:5173",
    trace: "on-first-retry",
  },
  webServer: {
    command: "npm run dev -- --host 127.0.0.1",
    url: "http://127.0.0.1:5173",
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
  ],
});
```

Now go back to repo root and create workflow folder:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
mkdir .github
mkdir .github\workflows
```

Create `.github/workflows/day-44-frontend-ci.yml`:

```yaml
name: Day 44 Frontend CI

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  frontend-ci:
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: day-44-github-actions-ci

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Setup Node
        uses: actions/setup-node@v6
        with:
          node-version: lts/*
          cache: npm
          cache-dependency-path: day-44-github-actions-ci/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run Vitest tests
        run: npm run test:run

      - name: Build React app
        run: npm run build

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - name: Upload Playwright report
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v5
        with:
          name: day-44-playwright-report
          path: day-44-github-actions-ci/playwright-report/
          retention-days: 30
```

Important: workflow files must be inside the repo root `.github/workflows`, not inside only the day folder.

Run locally before pushing:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey\day-44-github-actions-ci"
npm ci
npm run test:run
npm run build
npx playwright test
```

If `npm ci` fails because `package-lock.json` is missing, run:

```powershell
npm install
```

Then commit `package-lock.json`.

Commit:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
git add .
git commit -m "Day 44 add GitHub Actions CI for React tests"
```

Day 44 is complete when you can explain: CI runs checks automatically on GitHub so broken tests, failed builds, or broken browser flows are caught before code is merged.
