# Playwright — UI and API Test Automation from Scratch to Expert

> Subject #3 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: TypeScript (file 02). Estimated time: ~70 hours.
> Target outcome: Design, build, and run a maintainable Playwright UI + API test suite with CI, visual diffs, traces, and parallelism.

---

## 1. Overview & Learning Outcomes

### Why Playwright?

**Playwright vs. Alternatives:**
- **Selenium:** Older, slower, no auto-waiting, more brittle.
- **Cypress:** Fast for UI, limited to single domain, weak API testing, Node.js only.
- **Playwright:** Auto-wait, web-first assertions, cross-browser (Chromium, Firefox, WebKit), parallel execution, trace viewer, strong API support, TypeScript-first.

**Key Strengths:**
- **Auto-waiting:** Waits for actionability (visible, stable, enabled) before clicking—reduces flakes.
- **Web-first assertions:** `expect(locator).toBeVisible()` waits up to 5s by default.
- **Multi-browser & multi-device:** Chrome, Firefox, Safari; mobile emulation out-of-box.
- **Trace Viewer:** Record + replay test with DOM snapshots, network, screenshots.
- **Parallelism:** Automatic worker pool, sharding, fast CI.
- **Codegen:** `npx playwright codegen` generates tests by recording user actions.
- **Strong API support:** `page.request` for mocking, intercepting, and testing REST endpoints.

### Learning Outcomes (by end of module)

1. Write maintainable Playwright tests in TypeScript using page-object models.
2. Automate complex UI workflows: login, form submission, multi-step journeys.
3. Master locators (getByRole, getByTestId, filtering) and assertions (web-first).
4. Design data-driven and parametrized test suites.
5. Mock & intercept network traffic; perform API testing.
6. Combine UI + API testing (seed via API, verify UI).
7. Set up visual regression testing with snapshot management.
8. Configure Playwright for CI/CD with parallelism, sharding, and reporting.
9. Debug flaky tests using traces, videos, and replay.
10. Build a production-ready test framework with custom fixtures and architecture.

---

## 2. Detailed Syllabus (modules with hours)

| Module | Hours | Topics |
|--------|-------|--------|
| 1. Setup & Fundamentals | 5 | Install, config, first test, browser/page/context lifecycle |
| 2. Locators & Assertions | 8 | Locator strategies, web-first assertions, best practices |
| 3. Actions & Interactions | 6 | Click, fill, type, upload, drag, keyboard, form handling |
| 4. Auto-waiting & Waits | 4 | Actionability, timeouts, `waitFor`, flake reduction |
| 5. Configuration & Projects | 5 | multi-browser, mobile, baseURL, retries, reporters |
| 6. Fixtures & Test Setup | 6 | Built-in fixtures, custom fixtures, worker-scoped |
| 7. Page Object Model | 8 | Design patterns, composition, typed selectors, anti-patterns |
| 8. Authentication & State | 5 | Storage state, auth fixtures, session reuse, programmatic login |
| 9. Test Data & Parametrization | 5 | Data-driven tests, fixtures, parallel describe, seed data |
| 10. Network & Mocking | 6 | Route, fulfill, abort, HAR, capturing requests, intercepting |
| 11. API Testing | 7 | `request` fixture, CRUD, auth, JSON schema, contract smoke |
| 12. Mixed UI + API Tests | 4 | Seeding, verification, API cleanup, workflows |
| 13. Visual Regression | 5 | Screenshots, masking, updating snapshots, CI integration |
| 14. Accessibility Testing | 3 | axe-core, integration, reporting |
| 15. Debugging & Traces | 4 | Trace viewer, videos, `page.pause()`, PWDEBUG, codegen |
| 16. Component Testing | 2 | Playwright CT basics, scope and use cases |
| 17. Parallelism & Sharding | 5 | Workers, sharding, isolation, performance math |
| 18. CI/CD Integration | 6 | GitHub Actions, matrix, Docker, artifact management |
| 19. Reporters & Parsing | 4 | HTML, JUnit, blob, JSON; merging; integration |
| 20. Flaky Tests & Retry | 4 | Identification, policies, test.fixme/skip/slow, dashboards |
| 21. Advanced Patterns | 5 | Builder pattern, custom assertions, performance testing |
| **Total** | **~70** | |

---

## 3. Topics — Explanations with Examples

### Topic 1: Installation & Project Initialization

#### Concept
Playwright requires Node.js (v16+). Setup scaffolds a project with browser binaries, config, and example tests.

#### Example: TypeScript Setup
```typescript
// Step 1: Create project
// $ npm init playwright@latest --typescript

// Step 2: Install dependencies (auto-added):
// "devDependencies": {
//   "@playwright/test": "^1.47.0",
//   "@types/node": "^20.0.0",
//   "typescript": "^5.0.0"
// }

// Step 3: project structure
// playwright.config.ts       <- main config
// tests/example.spec.ts      <- example test
// tests-examples/            <- helper patterns
// playwright-report/         <- HTML report (after run)
```

#### Key Pitfalls
- **Missing browser binaries:** Run `npx playwright install` after install.
- **Mixing Jest + Playwright:** Playwright has its own test runner; don't mix with Jest/Mocha.
- **baseURL set but not used:** Config `baseURL: 'http://localhost:3000'` but `page.goto('/path')` still fails if server down.

---

### Topic 2: Configuration (playwright.config.ts)

#### Concept
Central config defines browsers, devices, reporters, timeout, retries, and project-specific settings.

#### Example: Multi-Browser + Mobile
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // Global timeout for entire test (default: no limit)
  timeout: 30 * 1000,
  
  // Expect() timeout (e.g., toBeVisible)
  expect: {
    timeout: 5000,
  },

  // Retries on CI only
  retries: process.env.CI ? 2 : 0,

  // Max failures before stopping
  maxFailures: 5,

  // Run tests in parallel
  workers: process.env.CI ? 1 : undefined, // 1 on CI, auto on local

  // Base URL for page.goto('/path')
  baseURL: 'http://localhost:3000',

  // Reporters
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results.json' }],
    process.env.CI ? ['github'] : ['list'],
  ],

  // Global setup/teardown
  globalSetup: './tests/global-setup.ts',
  globalTeardown: './tests/global-teardown.ts',

  // Projects: browsers, devices, config overrides
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  // Use for all projects
  use: {
    // Collect trace on first retry
    trace: 'on-first-retry',
    // Screenshot on failure
    screenshot: 'only-on-failure',
    // Video on failure
    video: 'retain-on-failure',
  },

  // Web server auto-start before tests
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

#### Key Pitfalls
- **Not setting baseURL:** Forces full URLs in tests; makes env switching harder.
- **Too many browsers locally:** 5 projects × tests = slow feedback; use 1 locally, 5 in CI.
- **Retries without analysis:** Retry all tests equally; classify flaky ones first.

---

### Topic 3: First Test Anatomy

#### Concept
A Playwright test is an async function with lifecycle: setup → page navigation → actions → assertions → teardown.

#### Example: Login + Verification
```typescript
import { test, expect } from '@playwright/test';

test('user logs in successfully', async ({ page }) => {
  // Arrange: navigate to app
  await page.goto('/login');

  // Act: fill form and submit
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: /sign in/i }).click();

  // Assert: verify redirect and user state
  await expect(page).toHaveURL('/dashboard');
  await expect(page.getByText(/welcome, user/i)).toBeVisible();
});

test.describe('authentication', () => {
  test.beforeEach(async ({ page }) => {
    // Before each test in group: navigate to login
    await page.goto('/login');
  });

  test('invalid password shows error', async ({ page }) => {
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('wrong');
    await page.getByRole('button', { name: /sign in/i }).click();
    
    await expect(page.getByText(/invalid credentials/i)).toBeVisible();
  });

  test('empty email shows validation', async ({ page }) => {
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: /sign in/i }).click();
    
    await expect(page.getByText(/email is required/i)).toBeVisible();
  });
});
```

#### Key Pitfalls
- **Missing `await`:** Promises not awaited = test moves on before action completes.
- **Not using `expect()` waits:** `expect().toBeVisible()` waits; manual checks don't.
- **Mixing UI and API without isolation:** Tests share state; need reset or fixtures.

---

### Topic 4: Locators — Deep Dive

#### Concept
Locators are queries that find elements reliably. Playwright's locator strategies prioritize accessibility (role, label, text) over brittle CSS/XPath.

#### Example: Locator Strategies
```typescript
import { test, expect } from '@playwright/test';

test('all locator strategies', async ({ page }) => {
  await page.goto('/register');

  // 1. getByRole: Most recommended. Use ARIA roles (button, textbox, etc.)
  await page.getByRole('button', { name: /submit/i }).click();
  
  // 2. getByLabel: For form inputs with <label>
  await page.getByLabel('First Name').fill('John');
  
  // 3. getByPlaceholder: For placeholder text
  await page.getByPlaceholder('Enter email').fill('john@example.com');
  
  // 4. getByText: For elements containing text (flexible regex)
  await expect(page.getByText(/welcome/i)).toBeVisible();
  
  // 5. getByTestId: For data-testid attribute (most stable for custom elements)
  await page.getByTestId('submit-btn').click();
  
  // 6. locator() with CSS selector
  const element = page.locator('css=.submit-button');
  
  // 7. locator() with XPath (escape braces for cross-browser)
  const xpath = page.locator('xpath=//button[contains(text(), "Submit")]');
  
  // Chaining & filtering
  const rows = page.locator('tr');
  const userRow = rows.filter({ hasText: 'John Doe' });
  await userRow.getByRole('button', { name: /edit/i }).click();
  
  // Count
  const count = await page.getByRole('listitem').count();
  expect(count).toBe(5);
  
  // Visible locators only
  const visibleItems = page.getByRole('listitem').filter({ visible: true });
  
  // Locator methods
  const label = page.getByLabel('Email');
  const value = await label.inputValue(); // For inputs
  const text = await page.getByText('Save').textContent();
  const isVisible = await element.isVisible();
  const isEnabled = await element.isEnabled();
});

test('avoid brittle selectors', async ({ page }) => {
  // BAD: depends on structure and styling
  // await page.locator('body > div > div > button:nth-child(3)').click();
  
  // GOOD: intent-based
  await page.getByRole('button', { name: /delete/i }).click();
  
  // BAD: magic strings
  // await page.locator('.btn-primary').click();
  
  // GOOD: semantic or data-testid
  await page.getByTestId('primary-action').click();
});
```

#### Key Pitfalls
- **XPath/CSS when role/label available:** Less maintainable; prone to break on refactor.
- **Not using locator filtering:** Leads to `.nth(0)` hacks; use `.filter()` instead.
- **getByText() too broad:** Matches partial text; use regex or exact match.
- **Mixing locator and direct selectors:** Inconsistency; standardize on one approach per project.

---

### Topic 5: Actions & Interactions

#### Concept
Actions simulate user behavior: click, type, select, drag, upload. Auto-waiting ensures element is ready.

#### Example: Comprehensive Actions
```typescript
import { test } from '@playwright/test';

test('all actions', async ({ page }) => {
  await page.goto('/form');

  // Click (auto-waits for visible + enabled)
  await page.getByRole('button', { name: /next/i }).click();
  
  // Double-click
  await page.locator('.editable-cell').dblclick();
  
  // Right-click
  await page.getByText('Options').click({ button: 'right' });
  
  // Fill (clears then types; for input/textarea)
  await page.getByLabel('Username').fill('testuser');
  
  // Type (character-by-character; slower, mimics typing speed)
  await page.getByLabel('Password').type('pass123', { delay: 100 });
  
  // Press keys (single or combo)
  await page.press('input[type=password]', 'Enter');
  await page.press('input', 'Control+A'); // Select all
  await page.press('input', 'Backspace');
  
  // Check/uncheck checkbox
  await page.getByRole('checkbox', { name: /agree/i }).check();
  await page.getByRole('checkbox', { name: /newsletter/i }).uncheck();
  
  // Select option from <select>
  await page.getByLabel('Country').selectOption('USA');
  await page.getByLabel('Options').selectOption(['opt1', 'opt2']); // Multiple
  
  // Hover (for dropdown triggers, tooltips)
  await page.getByRole('button', { name: /menu/i }).hover();
  
  // Drag and drop
  const source = page.getByText('Item 1');
  const target = page.getByText('Drop here');
  await source.dragTo(target);
  
  // File upload
  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles('./fixtures/test.pdf');
  await fileInput.setInputFiles([]); // Clear
  
  // File download
  const downloadPromise = page.waitForEvent('download');
  await page.getByRole('button', { name: /download/i }).click();
  const download = await downloadPromise;
  await download.saveAs('./downloads/file.pdf');
  
  // Scroll
  await page.getByText('Bottom Content').scrollIntoViewIfNeeded();
  await page.evaluate(() => window.scrollBy(0, 1000));
  
  // Focus
  await page.getByLabel('Email').focus();
  
  // Force action (skip actionability checks; use sparingly)
  await page.getByRole('button').click({ force: true });
});

test('form submission with delay', async ({ page }) => {
  await page.goto('/form');

  // Fill form
  const form = page.locator('form');
  await form.getByLabel('Name').fill('Alice');
  await form.getByLabel('Email').fill('alice@example.com');
  
  // Submit and wait for navigation
  await Promise.all([
    page.waitForNavigation(),
    form.getByRole('button', { name: /submit/i }).click(),
  ]);
  
  // Or wait for specific event
  const responsePromise = page.waitForResponse(res => 
    res.url().includes('/api/submit') && res.status() === 201
  );
  await form.getByRole('button', { name: /submit/i }).click();
  const response = await responsePromise;
});
```

#### Key Pitfalls
- **Using `click({ force: true })`:** Masks actionability issues; fix locator instead.
- **Mixing `fill()` and `type()`:** They behave differently (clear vs. append); be intentional.
- **File upload without `waitForEvent('download')`:** Loses file reference.
- **Not handling async uploads/submissions:** Use `waitForNavigation()` or `waitForResponse()`.

---

### Topic 6: Web-First Assertions

#### Concept
Assertions wait for conditions (default 5s) instead of failing immediately, reducing flakes.

#### Example: Web-First Assertions
```typescript
import { test, expect } from '@playwright/test';

test('web-first assertions', async ({ page }) => {
  await page.goto('/app');

  // Visibility (waits for visible)
  await expect(page.getByText('Welcome')).toBeVisible();
  await expect(page.getByRole('dialog')).not.toBeVisible();
  
  // Text content (waits for exact or regex)
  await expect(page.getByRole('heading')).toHaveText('Dashboard');
  await expect(page.getByText(/items?/i)).toHaveText(/\d+ items?/); // Regex
  
  // Partial text match
  await expect(page.getByRole('status')).toContainText('Loading');
  
  // Input value
  const input = page.getByLabel('Email');
  await input.fill('user@example.com');
  await expect(input).toHaveValue('user@example.com');
  
  // URL
  await page.goto('/products');
  await expect(page).toHaveURL('/products');
  await expect(page).toHaveURL(/products/); // Regex
  
  // Element count
  await expect(page.getByRole('listitem')).toHaveCount(3);
  await expect(page.getByRole('listitem')).toHaveCount(0); // Empty list
  
  // CSS class
  await expect(page.getByRole('button')).toHaveClass(/active/);
  
  // Attribute value
  await expect(page.locator('a')).toHaveAttribute('href', '/home');
  
  // Enabled/disabled
  await expect(page.getByRole('button', { name: /submit/i })).toBeEnabled();
  await expect(page.getByRole('button', { name: /disabled/i })).toBeDisabled();
  
  // Checked checkbox
  const checkbox = page.getByRole('checkbox');
  await expect(checkbox).toBeChecked();
  await expect(checkbox).not.toBeChecked();
  
  // Focused
  await input.focus();
  await expect(input).toBeFocused();
  
  // Page/locator state (convenience matchers)
  await expect(page).toHaveTitle(/Dashboard/);
  
  // Screenshot comparison (visual regression)
  await expect(page).toHaveScreenshot('dashboard.png');
  
  // Custom wait condition (if no built-in matcher)
  const startTime = Date.now();
  await expect(async () => {
    const count = parseInt(await page.getByText(/\d+/).textContent() || '0');
    expect(count).toBeGreaterThan(10);
  }).toPass();
});

test('assertion timeouts', async ({ page }) => {
  await page.goto('/slow-loader');

  // Override timeout for this assertion
  await expect(page.getByText('Loaded')).toBeVisible({ timeout: 10000 });
  
  // Global timeout (in config expect.timeout)
  await expect(page.getByRole('button')).toBeEnabled();
});

test('soft assertions (don\'t stop test)', async ({ page }) => {
  await page.goto('/app');

  // Soft assertion: continues even if fails
  await expect.soft(page.getByText('User')).toBeVisible();
  await expect.soft(page.getByText('Balance')).toHaveText('$100');
  
  // All soft failures reported at end
  console.log('Test continues after soft failures');
});
```

#### Key Pitfalls
- **Expecting immediate failures:** Tests wait 5s by default; seems slow on fast UIs.
- **Using sync checks:** `await page.isVisible()` returns boolean, no wait; use `expect()` instead.
- **Flaky screenshot comparisons:** Different OS/fonts render differently; use masking or update baselines.

---

### Topic 7: Auto-Waiting & Actionability

#### Concept
Playwright auto-waits for actions (click, fill) and assertions (expect). Actionability checks: visible, stable, enabled, non-obstructed.

#### Example: Auto-Waiting Behavior
```typescript
import { test, expect } from '@playwright/test';

test('auto-waiting for actions', async ({ page }) => {
  await page.goto('/async-loader');

  // Button hidden initially; auto-waits up to 30s (timeout)
  // Visible after 2s via JS; Playwright waits and clicks successfully
  await page.getByRole('button', { name: /load/i }).click();
  
  // Assert: list appears after 1s; expect() waits up to 5s
  await expect(page.getByRole('listitem')).toHaveCount(5);
});

test('manual waits when needed', async ({ page }) => {
  await page.goto('/app');

  // If condition not observable via UI (e.g., XHR pending), use manual wait
  await page.waitForFunction(() => {
    const val = (window as any).appState?.isReady;
    return val === true;
  }, { timeout: 10000 });
  
  // Or wait for network to be idle
  await page.waitForLoadState('networkidle');
  
  // Or wait for specific selector (legacy; prefer expect)
  await page.waitForSelector('.loaded-content', { timeout: 5000 });
});

test('non-actionable element handling', async ({ page }) => {
  await page.goto('/overlay');

  // Button behind overlay: Playwright fails with actionability error
  // Try to click
  try {
    await page.getByRole('button', { name: /submit/i }).click();
  } catch (e) {
    console.log('Error: Element is not actionable due to overlay');
    
    // Close overlay first
    await page.keyboard.press('Escape');
    
    // Now click succeeds
    await page.getByRole('button', { name: /submit/i }).click();
  }
});

test('explicitly wait for stable (CSS animation)', async ({ page }) => {
  await page.goto('/animated');

  const button = page.getByRole('button', { name: /fade in/i });
  
  // Auto-wait checks visibility and stability before action
  // For animations, may need explicit timeout
  await expect(button).toBeVisible();
  
  // Wait a bit for CSS animation to finish
  await page.waitForTimeout(1000);
  
  // Now click
  await button.click();
});

test('disabled input handling', async ({ page }) => {
  await page.goto('/form-workflow');

  const submitBtn = page.getByRole('button', { name: /submit/i });
  
  // Button disabled; fill form to enable it
  await expect(submitBtn).toBeDisabled();
  
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('pass');
  
  // Wait for button to enable
  await expect(submitBtn).toBeEnabled();
  
  // Now click
  await submitBtn.click();
});

test('force click when absolutely necessary', async ({ page }) => {
  await page.goto('/tricky-element');

  // Only use force: true if standard actionability fails and you understand why
  // Logs warning; should be exception, not norm
  await page.getByRole('button').click({ force: true });
});
```

#### Key Pitfalls
- **Overusing `waitForTimeout()`:** Static delays are slow; prefer observable conditions.
- **Not understanding actionability:** Click fails mysteriously; check element visibility/obstruction.
- **Using `force: true` as default:** Hides real issues; fix selector or DOM instead.

---

### Topic 8: Fixtures — Built-In & Custom

#### Concept
Fixtures provide reusable test dependencies (page, context, browser) with setup/teardown. Can be extended for custom needs (DB, auth, test data).

#### Example: Custom Fixtures
```typescript
import { test as base, expect, Page } from '@playwright/test';
import { Database } from './db';
import { AuthService } from './auth';

// Define custom fixture types
type TestFixtures = {
  db: Database;
  authenticatedPage: Page; // Page with logged-in state
  testDataId: string;
};

// Extend base test with custom fixtures
export const test = base.extend<TestFixtures>({
  // Custom fixture: Database connection
  db: async ({}, use) => {
    const db = new Database(':memory:');
    await db.init();
    
    // Setup
    console.log('DB initialized');
    
    // Provide to test
    await use(db);
    
    // Teardown
    await db.close();
    console.log('DB closed');
  },

  // Custom fixture: Authenticated page
  authenticatedPage: async ({ page, db }, use) => {
    // Setup: clear cookies, navigate to app
    await page.context().clearCookies();
    await page.goto('/');
    
    // Programmatically log in (via API, faster than UI)
    const authService = new AuthService();
    const token = await authService.login('testuser@example.com', 'password');
    
    // Set token in localStorage
    await page.evaluate(tk => {
      localStorage.setItem('authToken', tk);
    }, token);
    
    // Navigate to app (now authenticated)
    await page.goto('/dashboard');
    await expect(page.getByText(/welcome/i)).toBeVisible();
    
    // Provide to test
    await use(page);
    
    // Teardown (optional; context auto-closes)
  },

  // Custom fixture: Generate unique test data
  testDataId: async ({}, use) => {
    const id = `test-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    await use(id);
  },
});

// Test using custom fixtures
test('user creates project with authenticated page', async ({ authenticatedPage, testDataId, db }) => {
  const page = authenticatedPage;
  const projectName = `Project ${testDataId}`;
  
  // Already logged in
  await page.getByRole('button', { name: /new project/i }).click();
  
  await page.getByLabel('Project Name').fill(projectName);
  await page.getByRole('button', { name: /create/i }).click();
  
  // Verify in DB
  const project = await db.query('SELECT * FROM projects WHERE name = ?', [projectName]);
  expect(project).toBeDefined();
  
  // Verify in UI
  await expect(page.getByText(projectName)).toBeVisible();
});

// Worker-scoped fixture (initialized once per worker, shared across tests)
type WorkerFixtures = {
  server: any; // Shared app server
};

export const testWithServer = base.extend<TestFixtures & WorkerFixtures>({
  server: [
    async ({}, use) => {
      const server = await startServer();
      console.log('Server started on port 3000');
      
      await use(server);
      
      await server.close();
      console.log('Server stopped');
    },
    { scope: 'worker' }, // Scope: test (default) or worker
  ],

  db: async ({}, use) => {
    const db = new Database(':memory:');
    await db.init();
    await use(db);
    await db.close();
  },
});

// Test using worker-scoped fixture
testWithServer('multiple tests share one server', async ({ page, server }) => {
  await page.goto(`http://localhost:${server.port}`);
  await expect(page).toHaveTitle(/App/);
});
```

#### Key Pitfalls
- **Not isolating fixtures:** DB state leaks between tests; always reset in setup.
- **Fixture ordering complexity:** Deep dependencies hard to reason about; keep fixture tree shallow.
- **Worker-scoped fixtures for user-specific data:** Each worker gets one instance; tests may interfere.

---

### Topic 9: Page Object Model (POM) Pattern

#### Concept
POM encapsulates page structure and actions into reusable classes, improving maintainability and reducing duplication.

#### Example: POM Implementation
```typescript
// pages/base.page.ts
import { Page, Locator } from '@playwright/test';

export class BasePage {
  protected page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  async goto(path: string = '/'): Promise<void> {
    await this.page.goto(path);
  }

  async waitForPageReady(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }
}

// pages/login.page.ts
import { expect, Page } from '@playwright/test';
import { BasePage } from './base.page';

export class LoginPage extends BasePage {
  // Locators (getters for readability)
  private get emailInput(): Locator {
    return this.page.getByLabel('Email');
  }

  private get passwordInput(): Locator {
    return this.page.getByLabel('Password');
  }

  private get rememberMeCheckbox(): Locator {
    return this.page.getByRole('checkbox', { name: /remember me/i });
  }

  private get signInButton(): Locator {
    return this.page.getByRole('button', { name: /sign in/i });
  }

  private get errorMessage(): Locator {
    return this.page.getByRole('alert');
  }

  // Actions (user-centric methods)
  async login(email: string, password: string, rememberMe: boolean = false): Promise<void> {
    await this.goto('/login');
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    
    if (rememberMe) {
      await this.rememberMeCheckbox.check();
    }
    
    await this.signInButton.click();
  }

  async loginWithoutWait(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.signInButton.click();
  }

  // Assertions (but prefer fixtures + expect in tests)
  async verifyErrorShown(errorText: string): Promise<void> {
    await expect(this.errorMessage).toContainText(errorText);
  }

  async verifyPageLoaded(): Promise<void> {
    await expect(this.page).toHaveTitle(/Login/);
    await expect(this.emailInput).toBeVisible();
  }
}

// pages/dashboard.page.ts
export class DashboardPage extends BasePage {
  private get welcomeMessage(): Locator {
    return this.page.getByRole('heading', { name: /welcome/i });
  }

  private get logoutButton(): Locator {
    return this.page.getByRole('button', { name: /logout/i });
  }

  async logout(): Promise<void> {
    await this.logoutButton.click();
  }

  async verifyLoggedIn(userName: string): Promise<void> {
    await expect(this.welcomeMessage).toContainText(userName);
  }
}

// tests/auth.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/login.page';
import { DashboardPage } from './pages/dashboard.page';

test.describe('Authentication Flow', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
  });

  test('successful login', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123');
    
    // Verify navigation
    await expect(page).toHaveURL('/dashboard');
    
    // Verify logged-in state
    await dashboardPage.verifyLoggedIn('User');
  });

  test('invalid credentials show error', async ({ page }) => {
    await loginPage.login('user@example.com', 'wrongpassword');
    
    await loginPage.verifyErrorShown('Invalid credentials');
  });
});

// POM Anti-patterns to avoid
export class BadLoginPage extends BasePage {
  // BAD: Returns locators (test must know selector details)
  getEmailInput(): Locator {
    return this.page.locator('input[type="email"]');
  }

  // BAD: Mixes assertions into actions
  async login(email: string, password: string): Promise<void> {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button').click();
    
    // Don't assert here; let test decide
    await expect(this.page).toHaveURL('/dashboard');
  }

  // BAD: Generic methods that do everything
  async fillAndSubmitForm(data: any): Promise<void> {
    // Too flexible, harder to understand intent
  }
}

// GOOD: Composition over inheritance
class CreateProjectPage {
  constructor(private page: Page) {}

  async createProject(name: string, description: string): Promise<void> {
    await this.page.getByLabel('Project Name').fill(name);
    await this.page.getByLabel('Description').fill(description);
    await this.page.getByRole('button', { name: /create/i }).click();
  }
}

class ProjectListPage {
  constructor(private page: Page) {}

  async selectProject(name: string): Promise<CreateProjectPage> {
    await this.page.getByText(name).click();
    return new CreateProjectPage(this.page);
  }
}
```

#### Key Pitfalls
- **Over-Engineering:** POM for simple tests adds bloat; use when 3+ tests share actions.
- **God Objects:** Pages doing too much; split into smaller, composable classes.
- **Assertions in POM:** Pages should expose data; tests should assert. Breaks SoC.
- **Returning Locators:** Tests then hardcode selector logic; encapsulate intent instead.

---

### Topic 10: Authentication & Session Management

#### Concept
Reuse logged-in state across tests to speed up suite. Strategies: storage state (cookies/localStorage), fixtures, API login.

#### Example: Auth Fixture & Storage State
```typescript
// auth.setup.ts - Global setup file
import { chromium, FullConfig } from '@playwright/test';
import path from 'path';

const authFile = path.join(__dirname, 'auth.json');

async function globalSetup(config: FullConfig) {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();

  // Navigate to app and log in
  await page.goto('http://localhost:3000/login');
  await page.getByLabel('Email').fill('testuser@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: /sign in/i }).click();

  // Wait for redirect to confirm login
  await page.waitForURL('/dashboard');

  // Save storage state (cookies + localStorage)
  await context.storageState({ path: authFile });

  await browser.close();
}

export default globalSetup;

// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  globalSetup: require.resolve('./auth.setup.ts'),

  use: {
    // Automatically load auth.json before each test
    storageState: 'auth.json',
  },

  projects: [
    {
      name: 'chromium',
      use: { 
        storageState: 'auth.json', // Load for all tests in this project
      },
    },
  ],
});

// tests/dashboard.spec.ts
import { test, expect } from '@playwright/test';

test('user views dashboard (logged in)', async ({ page }) => {
  // No login needed; auth.json loaded automatically
  await page.goto('/dashboard');
  
  await expect(page.getByText(/welcome/i)).toBeVisible();
});

test('user profile shows email', async ({ page }) => {
  await page.goto('/profile');
  
  await expect(page.getByText('testuser@example.com')).toBeVisible();
});

// ========== OR: Custom Auth Fixture ==========

import { test as base } from '@playwright/test';

type AuthFixtures = {
  authenticatedContext: BrowserContext;
};

const test = base.extend<AuthFixtures>({
  authenticatedContext: async ({ browser }, use) => {
    // Create new context
    const context = await browser.newContext();
    const page = await context.newPage();

    // Log in programmatically
    await page.goto('http://localhost:3000/api/auth/login', {
      method: 'POST',
      data: {
        email: 'testuser@example.com',
        password: 'password123',
      },
    });

    // Save context with auth
    const cookies = await context.cookies();
    const storage = await context.storageState();

    // Provide context to test
    await use(context);

    // Cleanup
    await context.close();
  },
});

// Use fixture
test('with custom auth fixture', async ({ browser, authenticatedContext }) => {
  const page = await authenticatedContext.newPage();
  await page.goto('/dashboard');
  
  await expect(page.getByText(/welcome/i)).toBeVisible();
});

// ========== API-First Login ==========

export const testWithApiAuth = base.extend<AuthFixtures>({
  authenticatedContext: async ({ browser }, use) => {
    const context = await browser.newContext();
    
    // 1. Log in via API (fast)
    const loginRes = await context.request.post('http://localhost:3000/api/login', {
      data: {
        email: 'testuser@example.com',
        password: 'password123',
      },
    });
    
    const { token } = await loginRes.json();
    
    // 2. Add auth header to all requests
    await context.setExtraHTTPHeaders({
      Authorization: `Bearer ${token}`,
    });
    
    // 3. Or set cookie if server uses session cookies
    const cookies = await loginRes.headers()['set-cookie'];
    if (cookies) {
      await context.addCookies([
        {
          name: 'sessionId',
          value: token,
          url: 'http://localhost:3000',
        },
      ]);
    }
    
    await use(context);
    await context.close();
  },
});

testWithApiAuth('fast login via API', async ({ authenticatedContext }) => {
  const page = await authenticatedContext.newPage();
  await page.goto('/dashboard');
  
  await expect(page.getByText(/welcome/i)).toBeVisible();
});
```

#### Key Pitfalls
- **Hardcoding credentials:** Use env vars or fixture files, never in code.
- **Storage state conflict:** Tests with `storageState` + those without can interfere; be explicit.
- **Token expiry during test:** Long tests may need re-auth mid-suite; refresh token if needed.
- **Not testing logout:** Auth feature includes cleanup; test end-to-end.

---

### Topic 11: Test Data & Parametrization

#### Concept
Data-driven tests run same logic against multiple inputs. Reduces code duplication and improves coverage.

#### Example: Data-Driven Tests
```typescript
import { test, expect } from '@playwright/test';

// Parametrized test (same test, multiple data sets)
const loginData = [
  { email: 'user1@example.com', password: 'pass123', description: 'valid user' },
  { email: 'user2@example.com', password: 'pass456', description: 'another valid user' },
];

for (const data of loginData) {
  test(`login with ${data.description}`, async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill(data.email);
    await page.getByLabel('Password').fill(data.password);
    await page.getByRole('button', { name: /sign in/i }).click();
    
    await expect(page).toHaveURL('/dashboard');
  });
}

// Cleaner: test.describe with parametrization
test.describe('form validation', () => {
  const testCases = [
    { input: '', expectedError: 'Email is required' },
    { input: 'notanemail', expectedError: 'Invalid email format' },
    { input: 'user@example.com', expectedError: null }, // Valid
  ];

  testCases.forEach(({ input, expectedError }) => {
    test(`email field with "${input}"`, async ({ page }) => {
      await page.goto('/form');
      await page.getByLabel('Email').fill(input);
      await page.getByRole('button', { name: /submit/i }).click();

      if (expectedError) {
        await expect(page.getByText(expectedError)).toBeVisible();
      } else {
        await expect(page).toHaveURL('/success');
      }
    });
  });
});

// Parallel test.describe (tests in group run in parallel, groups sequential)
test.describe.parallel('feature A', () => {
  test('test 1', async ({ page }) => {
    // runs in parallel with test 2, 3
  });

  test('test 2', async ({ page }) => {});
  test('test 3', async ({ page }) => {});
});

test.describe.serial('feature B', () => {
  // These tests run sequentially (e.g., setup → action → verify)
  test('setup data', async ({ page }) => {});
  test('action', async ({ page }) => {});
  test('verify', async ({ page }) => {});
});

// Fixture-based test data
import { test as base } from '@playwright/test';

type DataFixtures = {
  testUsers: typeof testUsers;
  testProducts: typeof testProducts;
};

const testUsers = [
  { id: 1, name: 'Alice', role: 'admin' },
  { id: 2, name: 'Bob', role: 'user' },
];

const testProducts = [
  { id: 101, name: 'Laptop', price: 999 },
  { id: 102, name: 'Mouse', price: 29 },
];

const test = base.extend<DataFixtures>({
  testUsers: async ({}, use) => {
    // Setup: insert into DB
    await db.insertUsers(testUsers);
    await use(testUsers);
    // Teardown: clean up
    await db.deleteUsers(testUsers.map(u => u.id));
  },

  testProducts: async ({}, use) => {
    await db.insertProducts(testProducts);
    await use(testProducts);
    await db.deleteProducts(testProducts.map(p => p.id));
  },
});

test('list all products', async ({ page, testProducts }) => {
  await page.goto('/products');
  
  for (const product of testProducts) {
    await expect(page.getByText(product.name)).toBeVisible();
  }
});

test.describe('user permissions', () => {
  testUsers.forEach(user => {
    test(`${user.name} with role ${user.role}`, async ({ page }) => {
      // Seed test data via API
      const res = await page.request.post('/api/seed-user', {
        data: user,
      });
      
      await page.goto('/dashboard');
      await expect(page.getByText(user.name)).toBeVisible();
    });
  });
});
```

#### Key Pitfalls
- **Shared state across iterations:** Each param run gets fresh context; seed data per run.
- **Too many parameters:** Many combinations explode test count; use critical paths.
- **Not isolating parametrized groups:** Tests in one param may interfere with another.

---

### Topic 12: Network Mocking & Interception

#### Concept
Intercept HTTP requests/responses to mock APIs, simulate failures, or capture traffic.

#### Example: Network Mocking
```typescript
import { test, expect } from '@playwright/test';

test('mock API response', async ({ page }) => {
  // Intercept requests to /api/users
  await page.route('**/api/users', (route) => {
    // Respond with mock data
    route.abort('blockedbyresponse');
    // Or
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' },
      ]),
    });
  });

  await page.goto('/users');
  
  // App makes real request, gets mocked response
  await expect(page.getByText('Alice')).toBeVisible();
  await expect(page.getByText('Bob')).toBeVisible();
});

test('simulate API error', async ({ page }) => {
  // Respond with 500 error
  await page.route('**/api/products', (route) => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Server error' }),
    });
  });

  await page.goto('/products');
  
  await expect(page.getByText(/error loading products/i)).toBeVisible();
});

test('abort network request', async ({ page }) => {
  // Block requests to tracking/analytics
  await page.route('**/analytics/**', (route) => route.abort());
  await page.route('**/tracking/**', (route) => route.abort('blockedbyresponse'));

  // Page loads but tracking calls fail silently
  await page.goto('/');
  await expect(page.getByRole('heading')).toBeVisible();
});

test('capture and inspect requests', async ({ page }) => {
  let capturedRequest: any;

  await page.route('**/api/submit', (route) => {
    // Inspect request
    capturedRequest = route.request();
    console.log('URL:', capturedRequest.url());
    console.log('Method:', capturedRequest.method());
    console.log('Headers:', capturedRequest.headers());
    console.log('Body:', capturedRequest.postDataJSON());

    // Respond normally or with modification
    route.fulfill({
      status: 200,
      body: JSON.stringify({ success: true }),
    });
  });

  await page.goto('/form');
  await page.getByLabel('Name').fill('Test');
  await page.getByRole('button', { name: /submit/i }).click();

  // Verify request was captured
  expect(capturedRequest).toBeDefined();
  expect(capturedRequest.postDataJSON()).toEqual(
    expect.objectContaining({ name: 'Test' })
  );
});

test('HAR (HTTP Archive) replay', async ({ page }) => {
  // Record HAR file on first run (requires --update-snapshots)
  // Then replay from HAR on subsequent runs (offline-safe)
  
  // Or programmatically use HAR
  const recordHAR = false; // Set true to record, false to replay
  
  if (recordHAR) {
    const context = await page.context();
    await context.routeFromHAR('./fixtures/api-calls.har', {
      url: '**/api/**',
      update: true, // Update HAR with real responses
    });
  } else {
    // Replay from HAR
    await page.context().routeFromHAR('./fixtures/api-calls.har', {
      url: '**/api/**',
    });
  }

  await page.goto('/api-based-page');
  await expect(page.getByText('Loaded from HAR')).toBeVisible();
});

test('wait for response and inspect', async ({ page }) => {
  // Wait for specific response
  const responsePromise = page.waitForResponse(
    res => res.url().includes('/api/users') && res.status() === 200
  );

  await page.goto('/users');
  
  const response = await responsePromise;
  const data = await response.json();
  
  expect(data).toEqual(expect.arrayContaining([
    expect.objectContaining({ name: 'Alice' }),
  ]));
});

test('conditional mocking based on request', async ({ page }) => {
  await page.route('**/api/data', (route) => {
    const request = route.request();
    
    // Different responses based on query param
    if (request.url().includes('type=premium')) {
      route.fulfill({
        status: 200,
        body: JSON.stringify({ features: ['premium'] }),
      });
    } else {
      route.fulfill({
        status: 200,
        body: JSON.stringify({ features: ['basic'] }),
      });
    }
  });

  await page.goto('/data?type=premium');
  await expect(page.getByText('premium')).toBeVisible();
});
```

#### Key Pitfalls
- **Over-mocking:** Mocking too much hides real integration issues; mock only what's slow/unstable.
- **Stale mocks:** Mocks don't auto-update when API changes; verify against real API periodically.
- **Route ordering:** Routes match first-registered; put specific routes before wildcards.

---

### Topic 13: API Testing with `request` Fixture

#### Concept
Playwright's `request` fixture enables testing REST APIs directly (CRUD, auth, contracts).

#### Example: API Testing
```typescript
import { test, expect } from '@playwright/test';

test.describe('API: CRUD operations', () => {
  const baseURL = 'http://localhost:3000/api';
  let createdId: string;

  test('POST: create user', async ({ request }) => {
    const response = await request.post(`${baseURL}/users`, {
      data: {
        name: 'John Doe',
        email: 'john@example.com',
        age: 30,
      },
    });

    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(201);

    const body = await response.json();
    createdId = body.id;
    
    expect(body).toEqual(
      expect.objectContaining({
        name: 'John Doe',
        email: 'john@example.com',
      })
    );
  });

  test('GET: retrieve user', async ({ request }) => {
    const response = await request.get(`${baseURL}/users/${createdId}`);

    expect(response.ok()).toBeTruthy();
    const body = await response.json();
    
    expect(body.id).toBe(createdId);
    expect(body.name).toBe('John Doe');
  });

  test('PUT: update user', async ({ request }) => {
    const response = await request.put(`${baseURL}/users/${createdId}`, {
      data: {
        age: 31,
      },
    });

    expect(response.ok()).toBeTruthy();
    const body = await response.json();
    expect(body.age).toBe(31);
  });

  test('DELETE: remove user', async ({ request }) => {
    const response = await request.delete(`${baseURL}/users/${createdId}`);

    expect(response.status()).toBe(204);
  });

  test('GET: user not found after delete', async ({ request }) => {
    const response = await request.get(`${baseURL}/users/${createdId}`);

    expect(response.status()).toBe(404);
  });
});

test.describe('API: Authentication', () => {
  test('login and get token', async ({ request }) => {
    const response = await request.post('http://localhost:3000/api/login', {
      data: {
        email: 'user@example.com',
        password: 'password123',
      },
    });

    expect(response.ok()).toBeTruthy();
    const { token } = await response.json();
    
    return token; // Use in next tests
  });

  test('request with auth header', async ({ request }) => {
    // Get token first
    const loginRes = await request.post('http://localhost:3000/api/login', {
      data: { email: 'user@example.com', password: 'password123' },
    });
    const { token } = await loginRes.json();

    // Use token in protected endpoint
    const response = await request.get('http://localhost:3000/api/profile', {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    expect(response.ok()).toBeTruthy();
    const profile = await response.json();
    expect(profile.email).toBe('user@example.com');
  });

  test('unauthenticated request fails', async ({ request }) => {
    const response = await request.get('http://localhost:3000/api/profile');

    expect(response.status()).toBe(401);
  });
});

test.describe('API: JSON Schema Validation', () => {
  test('response matches schema', async ({ request }) => {
    const response = await request.get('http://localhost:3000/api/users/1');

    expect(response.ok()).toBeTruthy();
    const body = await response.json();

    // Manual schema check
    expect(body).toHaveProperty('id');
    expect(body).toHaveProperty('name');
    expect(body).toHaveProperty('email');
    expect(typeof body.id).toBe('number');
    expect(typeof body.name).toBe('string');
    expect(typeof body.email).toBe('string');
  });

  test('response with ajv validation', async ({ request }) => {
    const Ajv = require('ajv');
    const ajv = new Ajv();

    const userSchema = {
      type: 'object',
      properties: {
        id: { type: 'number' },
        name: { type: 'string' },
        email: { type: 'string', format: 'email' },
      },
      required: ['id', 'name', 'email'],
    };

    const response = await request.get('http://localhost:3000/api/users/1');
    const body = await response.json();

    const validate = ajv.compile(userSchema);
    const valid = validate(body);

    expect(valid).toBeTruthy();
    if (!valid) {
      console.log('Validation errors:', validate.errors);
    }
  });
});

test.describe('API: Error Handling', () => {
  test('4xx error response', async ({ request }) => {
    const response = await request.get('http://localhost:3000/api/users/999');

    expect(response.status()).toBe(404);
    const body = await response.json();
    expect(body.error).toContain('not found');
  });

  test('5xx server error', async ({ request }) => {
    // Assume endpoint that triggers 500
    const response = await request.get('http://localhost:3000/api/error');

    expect(response.status()).toBe(500);
  });
});

test('multipart form submission', async ({ request }) => {
  const response = await request.post('http://localhost:3000/api/upload', {
    multipart: {
      file: {
        name: 'test.txt',
        mimeType: 'text/plain',
        buffer: Buffer.from('test content'),
      },
      description: 'Test upload',
    },
  });

  expect(response.ok()).toBeTruthy();
});
```

#### Key Pitfalls
- **Mixing response.text() and response.json():** Response body consumed once; cache result.
- **Not validating schemas:** API contracts drift; use schema validation (ajv, joi).
- **Hardcoded IDs in tests:** Use responses from previous tests or env-based setup.

---

### Topic 14: Mixed UI + API Testing

#### Concept
Combine UI and API testing: seed data via API (fast), verify in UI; perform actions in UI, verify with API calls.

#### Example: Integrated UI + API Tests
```typescript
import { test, expect } from '@playwright/test';

test.describe('UI + API Integration', () => {
  const baseURL = 'http://localhost:3000/api';
  let userId: string;

  test('seed user via API, verify in UI', async ({ request, page }) => {
    // Step 1: Create user via API (fast)
    const createRes = await request.post(`${baseURL}/users`, {
      data: {
        name: 'Alice',
        email: 'alice@example.com',
      },
    });
    const user = await createRes.json();
    userId = user.id;

    // Step 2: Load UI and verify user appears
    await page.goto('/users');
    await expect(page.getByText('Alice')).toBeVisible();
    await expect(page.getByText('alice@example.com')).toBeVisible();
  });

  test('perform action in UI, verify with API', async ({ request, page }) => {
    // Step 1: Perform action in UI (click, fill, submit)
    await page.goto('/users');
    await page.getByRole('button', { name: /edit/ }).first().click();
    await page.getByLabel('Email').fill('alice.new@example.com');
    await page.getByRole('button', { name: /save/i }).click();

    // Verify success message
    await expect(page.getByText(/saved successfully/i)).toBeVisible();

    // Step 2: Verify change with API
    const getRes = await request.get(`${baseURL}/users/${userId}`);
    const updatedUser = await getRes.json();
    
    expect(updatedUser.email).toBe('alice.new@example.com');
  });

  test('cleanup via API after UI test', async ({ request, page }) => {
    // Create multiple users via UI
    await page.goto('/users');
    await page.getByRole('button', { name: /add user/i }).click();
    await page.getByLabel('Name').fill('Bob');
    await page.getByLabel('Email').fill('bob@example.com');
    await page.getByRole('button', { name: /create/i }).click();

    // Verify in UI
    await expect(page.getByText('Bob')).toBeVisible();

    // Cleanup via API (faster than UI delete)
    const listRes = await request.get(`${baseURL}/users`);
    const users = await listRes.json();
    const bob = users.find((u: any) => u.email === 'bob@example.com');

    if (bob) {
      const deleteRes = await request.delete(`${baseURL}/users/${bob.id}`);
      expect(deleteRes.status()).toBe(204);
    }
  });

  test('login via API, perform actions in UI', async ({ request, page, context }) => {
    // Step 1: Log in via API and get token
    const loginRes = await request.post(`http://localhost:3000/api/login`, {
      data: { email: 'alice@example.com', password: 'password' },
    });
    const { token } = await loginRes.json();

    // Step 2: Set token in browser context
    await context.addCookies([
      {
        name: 'authToken',
        value: token,
        url: 'http://localhost:3000',
      },
    ]);

    // Step 3: Now UI tests run with auth
    await page.goto('/dashboard');
    await expect(page.getByText(/welcome/i)).toBeVisible();
  });

  test('data-driven workflow: seed multiple, verify', async ({ request, page }) => {
    // Seed 3 users via API
    const userEmails = ['user1@example.com', 'user2@example.com', 'user3@example.com'];
    
    for (const email of userEmails) {
      await request.post(`${baseURL}/users`, {
        data: { name: email.split('@')[0], email },
      });
    }

    // Verify all appear in UI
    await page.goto('/users');
    
    for (const email of userEmails) {
      await expect(page.getByText(email)).toBeVisible();
    }
  });
});
```

#### Key Pitfalls
- **Test order dependency:** API sets up state, UI test assumes it exists. Use fixtures/beforeEach.
- **Auth token expiry:** Long tests may need re-auth mid-run; refresh token in fixtures.
- **Network latency:** UI test starts immediately after API setup; may need poll/wait.

---

### Topic 15: Visual Regression Testing

#### Concept
Compare screenshots to detect unintended visual changes (layout, styling, colors).

#### Example: Visual Tests
```typescript
import { test, expect } from '@playwright/test';

test('homepage screenshot matches baseline', async ({ page }) => {
  await page.goto('/');
  
  // Waits for page ready, then takes screenshot
  await page.waitForLoadState('networkidle');
  
  // Compare against baseline (stored in __screenshots__ folder)
  await expect(page).toHaveScreenshot('homepage.png');
});

test('component visual regression', async ({ page }) => {
  await page.goto('/buttons');
  
  const button = page.getByRole('button', { name: /submit/i });
  
  // Screenshot single element, not full page
  await expect(button).toHaveScreenshot('button-default.png');
  
  // Hover state
  await button.hover();
  await expect(button).toHaveScreenshot('button-hover.png');
  
  // Disabled state
  await button.evaluate(el => (el as HTMLButtonElement).disabled = true);
  await expect(button).toHaveScreenshot('button-disabled.png');
});

test('responsive layout (mobile)', async ({ page }) => {
  // Emulate mobile viewport
  await page.setViewportSize({ width: 375, height: 667 });
  
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage-mobile.png');
});

test('screenshot with masking (ignore dynamic content)', async ({ page }) => {
  await page.goto('/dashboard');
  
  // Mask timestamp/date elements
  await expect(page).toHaveScreenshot('dashboard.png', {
    mask: [
      page.locator('[data-testid="timestamp"]'),
      page.locator('.dynamic-banner'),
    ],
    maskColor: '#808080', // Gray out masked areas
  });
});

test('screenshot with animated content', async ({ page }) => {
  await page.goto('/animations');
  
  // Wait for animation to complete
  await page.waitForFunction(() => {
    const el = document.querySelector('[data-testid="loading"]');
    return el && el.classList.contains('completed');
  });
  
  await expect(page).toHaveScreenshot('animation-complete.png');
});

test('update baselines with --update-snapshots', async ({ page }) => {
  // Run with: npx playwright test --update-snapshots
  // This overwrites baseline screenshots with new ones
  
  await page.goto('/new-design');
  await expect(page).toHaveScreenshot('new-design.png');
});

test('PDF export visual test', async ({ page }) => {
  await page.goto('/invoice/123');
  
  // Generate PDF and compare
  const pdf = await page.pdf({ path: 'invoice.pdf' });
  
  // Visual assertions on rendered PDF
  const screenshotBuffer = await page.screenshot();
  expect(screenshotBuffer.toString('base64')).toMatch(/data:image/);
});
```

#### Configuration for Visual Tests
```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    // Screenshot settings
    screenshot: 'only-on-failure',
    
    // On visual diff, update baseline (requires --update-snapshots flag)
    // snapshotDir: './screenshots',
    // snapshotPathTemplate: '{dir}/{testFileDir}/{testFileName}-{arg}{ext}',
  },

  reporter: [
    [
      'html',
      {
        // HTML report includes screenshot diffs
        outputFolder: 'html-report',
      },
    ],
  ],

  projects: [
    {
      name: 'chromium',
      use: {
        // Disable animations for consistent screenshots
        reducedMotion: 'reduce',
      },
    },
  ],
});
```

#### Key Pitfalls
- **Flaky baselines (font rendering, anti-aliasing):** Different OS/browser = different pixels. Use CI image/Docker.
- **Not masking dynamic content:** Timestamps, API data cause false diffs; mask aggressively.
- **Screenshot resolution:** Full-page shots very wide; use locator-based screenshots for components.

---

### Topic 16: Accessibility Testing

#### Concept
Integrated accessibility checks using axe-core plugin; catch WCAG violations early.

#### Example: Accessibility Tests
```typescript
import { test, expect } from '@playwright/test';
import { injectAxe, getViolations } from 'axe-playwright';

test('homepage has no accessibility violations', async ({ page }) => {
  await page.goto('/');
  
  // Inject axe and run scan
  await injectAxe(page);
  const violations = await getViolations(page);
  
  expect(violations).toHaveLength(0);
});

test('check specific region for a11y', async ({ page }) => {
  await page.goto('/form');
  
  await injectAxe(page);
  
  // Check only main content area
  const violations = await getViolations(page, {
    selector: 'main',
  });
  
  expect(violations).toHaveLength(0);
});

test('skip known violations (test failure)', async ({ page }) => {
  await page.goto('/legacy-page');
  
  await injectAxe(page);
  const violations = await getViolations(page, {
    // Ignore known issues for now (track separately)
    exclude: ['color-contrast', 'missing-alt-text'],
  });
  
  // Expect to fail; document why
  expect(violations.length).toBeGreaterThan(0);
  console.log('Known a11y issues:', violations);
});

test('form labels and ARIA attributes', async ({ page }) => {
  await page.goto('/form');
  
  // Verify form elements have labels
  const inputs = page.locator('input');
  const count = await inputs.count();
  
  for (let i = 0; i < count; i++) {
    const input = inputs.nth(i);
    const ariaLabel = await input.getAttribute('aria-label');
    const forAttr = await input.getAttribute('id');
    
    // Either aria-label or associated <label> with for=id
    const hasLabel = ariaLabel || forAttr;
    expect(hasLabel).toBeTruthy();
  }
  
  await injectAxe(page);
  const violations = await getViolations(page);
  expect(violations).toHaveLength(0);
});

test('keyboard navigation', async ({ page }) => {
  await page.goto('/interactive');
  
  // Tab through elements; all interactive elements reachable
  let tabCount = 0;
  while (tabCount < 20) { // Safety limit
    const focused = await page.evaluate(() => document.activeElement?.tagName);
    
    if (focused === 'BUTTON' || focused === 'A') {
      // Reachable via keyboard
      tabCount++;
    }
    
    await page.keyboard.press('Tab');
  }
  
  expect(tabCount).toBeGreaterThan(0);
});
```

#### Key Pitfalls
- **Only unit-testing a11y:** Integration testing catches real-world issues; run in full app.
- **False positives:** axe has some false flags; review violations context.
- **Dynamic content:** Run scan after content loads; use `injectAxe()` after navigation.

---

### Topic 17: Debugging & Traces

#### Concept
Playwright captures traces (DOM, network, screenshots) and offers debug tools (Trace Viewer, PWDEBUG, codegen).

#### Example: Debugging
```typescript
import { test, expect } from '@playwright/test';

test('example with debug pause', async ({ page }) => {
  await page.goto('/');
  
  // Pause execution; opens inspector in browser
  // Inspect DOM, run JS, step through
  await page.pause();
  
  await page.getByRole('button').click();
});

test('trace recording', async ({ page }) => {
  // Manually start trace (normally auto in config)
  await page.context().tracing.start({ screenshots: true, snapshots: true });
  
  // Run test
  await page.goto('/');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByRole('button').click();
  
  // Stop and save trace
  await page.context().tracing.stop({ path: 'trace.zip' });
  
  // Open: npx playwright show-trace trace.zip
});

test('video and screenshot recording', async ({ page }) => {
  // Video auto-recorded when configured
  // On failure, check: test-results/<test-name>/<browser>/video.webm
  
  await page.goto('/');
  await expect(page.getByText('Welcome')).toBeVisible();
});

// Env var debugging
// PWDEBUG=1 npx playwright test <- opens debug mode
// npx playwright codegen http://localhost:3000 <- record test steps
```

#### Key Pitfalls
- **Large trace files:** Traces with many snapshots are huge; enable selectively.
- **Debug mode slow:** PWDEBUG useful for development, not CI.
- **Screenshots vs. Snapshots:** Screenshots are images (visual), snapshots are DOM trees (for inspection).

---

### Topic 18: Configuration & Parallelism

#### Concept
Playwright parallelizes test execution across workers and shards for fast CI feedback.

#### Example: Parallelism Config
```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  // Workers: number of parallel processes (default: 1 on CI, auto on local)
  workers: process.env.CI ? 1 : undefined,

  // Sharding: split tests across machines
  shard: process.env.SHARD ? {
    current: parseInt(process.env.SHARD.split('/')[0]),
    total: parseInt(process.env.SHARD.split('/')[1]),
  } : undefined,

  // Fully parallel: all tests in all files run in parallel
  fullyParallel: true,

  projects: [
    {
      name: 'chromium',
      use: {},
    },
    {
      name: 'firefox',
      use: {},
    },
  ],
});

// Sharding in CI: distribute tests across 4 machines
// Run 1/4: SHARD=1/4 npx playwright test
// Run 2/4: SHARD=2/4 npx playwright test
// Run 3/4: SHARD=3/4 npx playwright test
// Run 4/4: SHARD=4/4 npx playwright test
```

#### Key Pitfalls
- **Shared state across shards:** Each shard may fail independently; isolate test data.
- **Database lock contention:** Too many workers on single DB = conflicts; use in-memory or sharded DBs.

---

### Topic 19: CI/CD Integration

#### Concept
Run Playwright in CI (GitHub Actions, GitLab CI, etc.) with parallelism, reporting, and artifact management.

#### Example: GitHub Actions
```yaml
# .github/workflows/test.yml
name: Playwright Tests

on: [push, pull_request]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Run tests (shard ${{ matrix.shard }}/4)
        run: npx playwright test --shard=${{ matrix.shard }}/4
      
      - name: Upload blob report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: blob-report-${{ matrix.shard }}
          path: blob-report
      
      - name: Merge reports
        if: always()
        run: npx playwright merge-reports --reporter html ./blob-report-*
      
      - name: Upload HTML report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: html-report
          path: playwright-report/
          retention-days: 30
      
      - name: Publish report
        if: always()
        uses: dawidd6/action-download-artifact@v2
        with:
          workflow: test.yml
          name: html-report
          path: report
      
      - name: Comment PR with report
        if: always() && github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `[Test Report](https://...)`
            })
```

#### Key Pitfalls
- **Artifact explosion:** 4 shards × 10 files = bloat. Use blob reports + merge.
- **Timeout misconfiguration:** CI timeout too short; add buffer for slow test runs.

---

## 4. Exercises

### Beginner

1. **Install & Smoke Test:** Install Playwright, run example test, record with codegen on playwright.dev.
2. **Locator Kata (5 tasks):** Given HTML snippets, write correct locators (role, label, testid).
3. **First API Test:** GET request to jsonplaceholder.typicode.com/users/1, assert response.

### Intermediate

1. **Page Object Model:** Build POM for saucedemo.com login + product listing; 5+ tests.
2. **Data-Driven Login:** Parametrize 3 user + password combinations; verify login success/failure.
3. **Network Mock:** Intercept `/api/products` endpoint; mock empty list; verify UI handles gracefully.
4. **Visual Regression:** Take baselines of 3 pages; modify CSS; verify diff detected.
5. **API CRUD:** Build Express service with GET/POST/PUT/DELETE /items; test full cycle with schema validation (ajv).

### Advanced

1. **Custom Auth Fixture:** Extend test with storageState fixture; parallel-safe; reuse across 10+ tests.
2. **Contract Testing:** Define OpenAPI schema; validate all API responses against spec.
3. **GitHub Actions CI:** Setup 4-way sharding, blob report merge, HTML publish.
4. **Flaky Test Detector:** Parse reporter JSON; identify tests with >20% failure rate.

### Capstone Project

**"Test Framework for Full-Stack App"**

Build and deliver a Playwright TS test suite for a demo app (saucedemo, conduit, or similar):

- **POM Layer:** 5+ page classes; login, cart, checkout flows.
- **API Layer:** Seed data via API; CRUD endpoint testing.
- **UI + API:** Login via API, perform checkout in UI, verify DB via API.
- **Visual Testing:** 3 page baselines with diff masking.
- **Data-Driven:** 10+ parametrized test cases.
- **CI/CD:** GitHub Actions with 4-way sharding, HTML+JUnit reports.
- **Debugging:** Trace + video on failure; README with architecture diagram.
- **Flaky-Test Policy:** Document how to handle retries, skip, fixme.

Deliverable: GitHub repo with:
- `README.md`: architecture, setup, running tests, CI/CD flow, flaky-test policy.
- `tests/` with POM, API, UI+API, visual, smoke, e2e specs.
- `playwright.config.ts` with multi-project, sharding, reporters.
- `.github/workflows/test.yml` for CI.
- `tests/fixtures/` with custom fixtures (auth, db, etc.).
- `docs/DESIGN.md`: test strategy, coverage matrix, known issues.

---

## 5. Additional Reading & Resources

### Official
- **[Playwright Docs](https://playwright.dev)** — authoritative reference.
- **[Trace Viewer](https://playwright.dev/docs/trace-viewer)** — debug guide.
- **[Test Configuration](https://playwright.dev/docs/test-configuration)** — config deep-dive.

### Courses & Tutorials
- **Debbie O'Brien's Playwright YouTube** — demo-heavy tutorials.
- **Microsoft Playwright Channel** — official release videos.
- **Test Automation University (Applitools)** — "Playwright with TypeScript" course.
- **Playwright by Example blog** — real-world patterns.

### Community
- **[awesome-playwright](https://github.com/mxschmitt/awesome-playwright)** — curated links, plugins, examples.
- **GitHub Discussions** — peer support.

### Related Tools
- **Axe-core** — accessibility scanning.
- **Allure** — advanced reporting.
- **@testing-library** — semantic query helpers (Playwright has built-in equivalents).

---

## 6. Index

| Topic | Section |
|-------|---------|
| Installation & Setup | 2, 3.1 |
| Configuration | 3.2 |
| Locators | 3.4 |
| Actions | 3.5 |
| Assertions | 3.6 |
| Auto-Waiting | 3.7 |
| Fixtures | 3.8 |
| Page Object Model | 3.9 |
| Authentication | 3.10 |
| Test Data & Parametrization | 3.11 |
| Network Mocking | 3.12 |
| API Testing | 3.13 |
| Mixed UI + API | 3.14 |
| Visual Regression | 3.15 |
| Accessibility | 3.16 |
| Debugging | 3.17 |
| CI/CD | 3.19 |

---

## 7. References

- Playwright 1.47+ (www.npmjs.com/package/@playwright/test)
- TypeScript 5+
- Node.js 16+
- GitHub Actions (optional, for CI examples)
- Docker (optional, for containerized runs)

---

**End of Study Guide**

Next Subject: Testing Frameworks & Test Structure (File 04).

Estimated total time: 70 hours of hands-on practice + reference reading.

Good luck with your Playwright mastery journey!
