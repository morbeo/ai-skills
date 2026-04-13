# Playwright Reference

## Setup

```bash
npm init playwright@latest
# or add to existing project:
npm install -D @playwright/test
npx playwright install
```

---

## Config (playwright.config.ts)

```ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
    testDir: "./e2e",
    timeout: 30_000,
    expect: { timeout: 5_000 },
    fullyParallel: true,
    forbidOnly: !!process.env.CI,
    retries: process.env.CI ? 2 : 0,
    reporter: [["html"], ["list"]],

    use: {
        baseURL: "http://localhost:3000",
        trace: "on-first-retry",
        screenshot: "only-on-failure",
    },

    projects: [
        { name: "chromium", use: { ...devices["Desktop Chrome"] } },
        { name: "firefox",  use: { ...devices["Desktop Firefox"] } },
        { name: "webkit",   use: { ...devices["Desktop Safari"] } },
        { name: "mobile",   use: { ...devices["iPhone 14"] } },
    ],

    webServer: {
        command: "npm run dev",
        url: "http://localhost:3000",
        reuseExistingServer: !process.env.CI,
    },
});
```

---

## Locator hierarchy (prefer top to bottom)

```ts
// 1. Role (most accessible, most stable)
page.getByRole("button", { name: /submit/i })
page.getByRole("textbox", { name: "Email" })
page.getByRole("heading", { level: 1 })

// 2. Label text
page.getByLabel("Password")

// 3. Placeholder
page.getByPlaceholder("Search…")

// 4. Text content
page.getByText("Welcome back")

// 5. Alt text (images)
page.getByAltText("Company logo")

// 6. Test ID (last resort — fragile but unambiguous)
page.getByTestId("submit-button")    // data-testid="submit-button"

// Avoid: CSS selectors, XPath (implementation details)
```

---

## Web-first assertions (auto-retry until timeout)

```ts
// ✅ web-first — retries automatically
await expect(page.getByRole("status")).toHaveText("Saved");
await expect(page.getByRole("button")).toBeEnabled();
await expect(page.getByRole("dialog")).toBeVisible();
await expect(page.getByRole("list")).toHaveCount(3);
await expect(page).toHaveURL("/dashboard");
await expect(page).toHaveTitle(/Dashboard/);

// ❌ avoid — evaluates immediately, flaky
expect(await page.isVisible(".spinner")).toBe(false);
```

---

## Fixtures

### Built-in fixtures

```ts
import { test, expect } from "@playwright/test";

test("login page", async ({ page, context, browser }) => {
    await page.goto("/login");
    // page:    new page per test
    // context: new browser context per test (isolated cookies/storage)
    // browser: shared browser instance
});
```

### Custom fixtures (Page Object Model)

```ts
// fixtures.ts
import { test as base } from "@playwright/test";
import { LoginPage } from "./pages/LoginPage";
import { DashboardPage } from "./pages/DashboardPage";

export const test = base.extend<{
    loginPage: LoginPage;
    dashboardPage: DashboardPage;
}>({
    loginPage: async ({ page }, use) => {
        await use(new LoginPage(page));
    },
    dashboardPage: async ({ page }, use) => {
        await use(new DashboardPage(page));
    },
});
export { expect } from "@playwright/test";
```

### Page Object Model

```ts
// pages/LoginPage.ts
import { type Page, type Locator } from "@playwright/test";

export class LoginPage {
    readonly emailInput: Locator;
    readonly passwordInput: Locator;
    readonly submitButton: Locator;

    constructor(private page: Page) {
        this.emailInput    = page.getByLabel("Email");
        this.passwordInput = page.getByLabel("Password");
        this.submitButton  = page.getByRole("button", { name: /sign in/i });
    }

    async goto() {
        await this.page.goto("/login");
    }

    async login(email: string, password: string) {
        await this.emailInput.fill(email);
        await this.passwordInput.fill(password);
        await this.submitButton.click();
    }
}
```

### Authenticated state fixture

```ts
// auth.setup.ts — runs once, saves cookies
import { test as setup } from "@playwright/test";

setup("authenticate", async ({ page }) => {
    await page.goto("/login");
    await page.getByLabel("Email").fill("user@test.com");
    await page.getByLabel("Password").fill("password");
    await page.getByRole("button", { name: "Sign in" }).click();
    await page.waitForURL("/dashboard");
    await page.context().storageState({ path: "e2e/.auth/user.json" });
});
```

```ts
// playwright.config.ts
projects: [
    { name: "setup", testMatch: /auth\.setup\.ts/ },
    {
        name: "authenticated",
        use: { storageState: "e2e/.auth/user.json" },
        dependencies: ["setup"],
    },
],
```

---

## Network mocking

```ts
// Mock an API endpoint
await page.route("**/api/products", route =>
    route.fulfill({
        status: 200,
        body: JSON.stringify([{ id: 1, name: "Widget" }]),
    })
);

// Intercept and modify response
await page.route("**/api/search*", async route => {
    const response = await route.fetch();
    const json = await response.json();
    json.results = [];
    await route.fulfill({ json });
});

// Abort a request (simulate network failure)
await page.route("**/api/flaky", route => route.abort());
```

---

## CI setup (GitHub Actions)

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7
```

---

## Debugging

```bash
npx playwright test --debug                # step through with inspector
npx playwright test --headed               # see the browser
npx playwright test --trace on             # record trace for every test
npx playwright show-trace trace.zip        # open trace viewer
npx playwright codegen http://localhost:3000   # record interactions → test code
```

---

## Common patterns

### Wait for navigation

```ts
await Promise.all([
    page.waitForURL("/dashboard"),
    page.getByRole("button", { name: "Submit" }).click(),
]);
```

### Handle dialogs

```ts
page.on("dialog", dialog => dialog.accept());
await page.getByRole("button", { name: "Delete" }).click();
```

### File upload

```ts
await page.getByLabel("Upload CSV").setInputFiles("tests/data/sample.csv");
```

### Download

```ts
const [download] = await Promise.all([
    page.waitForEvent("download"),
    page.getByRole("button", { name: "Export" }).click(),
]);
await download.saveAs("./tmp/export.csv");
```

### Keyboard / hover

```ts
await page.getByRole("textbox").press("Control+A");
await page.getByRole("menuitem", { name: "File" }).hover();
```
