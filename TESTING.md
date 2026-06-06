# WeatherOps Frontend - Testing Guide

## Testing Strategy

This document outlines the testing approach for the WeatherOps frontend application.

## Testing Pyramid

```
                    ▲
                   ╱ ╲
                  ╱ E2E ╲      (End-to-End Tests)
                 ╱       ╲     - User journeys
                ╱─────────╲    - Real browser
               ╱  UI Tests  ╲   - 5-10 tests
              ╱             ╲
             ╱───────────────╲
            ╱  Integration    ╲  (Component Tests)
           ╱      Tests        ╲ - Component behavior
          ╱                     ╲ - Mock dependencies
         ╱───────────────────────╲ 20-40 tests
        ╱      Unit Tests         ╲
       ╱  (Helper Functions)       ╲
      ╱   - Pure functions         ╲
     ╱    - Utilities              ╱
    ╱────────────────────────────╱
   ╱        Many Tests          ╱
  ╱─────────────────────────────╱

Test Distribution:
- Unit Tests (60%): Utilities, helpers, validators
- Integration Tests (30%): Component behavior
- E2E Tests (10%): Critical user journeys
```

## Test Setup

### Playwright Configuration

The project uses Playwright for E2E testing:

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  testMatch: '**/*.spec.ts',
  
  // Shared settings for all
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  
  // Configure projects
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
  ],
  
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## Running Tests

### Development Testing

```bash
# Run all E2E tests
npm run test:e2e

# Run specific test file
npm run test:e2e -- auth.spec.ts

# Run tests in headed mode (see browser)
npm run test:e2e:headed

# Run tests with Playwright UI
npm run test:e2e:ui

# Run specific test in debug mode
npm run test:e2e:debug -- locations.spec.ts

# Run tests in watch mode
npm run test:e2e -- --watch
```

### CI/CD Testing

```bash
# Full test suite (used in CI)
npm run test:e2e

# Generate test report
npm run test:e2e -- --reporter=html
# Open report: npx playwright show-report
```

---

## E2E Testing Examples

### 1. Authentication Tests

```typescript
// e2e/auth.spec.ts
import { test, expect, Page } from '@playwright/test';

test.describe('Authentication', () => {
  let page: Page;

  test.beforeEach(async ({ page: testPage }) => {
    page = testPage;
    // Navigate to login page
    await page.goto('/');
  });

  test('should display login form', async () => {
    // Check form elements exist
    await expect(page.locator('input[type="email"]')).toBeVisible();
    await expect(page.locator('input[type="password"]')).toBeVisible();
    await expect(page.locator('button:has-text("Login")')).toBeVisible();
  });

  test('should login successfully with valid credentials', async () => {
    // Fill form
    await page.locator('input[type="email"]').fill('test@example.com');
    await page.locator('input[type="password"]').fill('Test123!');

    // Submit form
    await page.locator('button:has-text("Login")').click();

    // Wait for navigation to overview page
    await page.waitForURL('**/overview');

    // Check page loaded
    await expect(page.locator('h1')).toContainText('Overview');
  });

  test('should show error on invalid credentials', async () => {
    // Fill with wrong credentials
    await page.locator('input[type="email"]').fill('wrong@example.com');
    await page.locator('input[type="password"]').fill('WrongPassword');

    // Submit form
    await page.locator('button:has-text("Login")').click();

    // Check error message
    await expect(
      page.locator('text=Invalid credentials')
    ).toBeVisible();
  });

  test('should navigate to register page', async () => {
    // Click register link
    await page.locator('a:has-text("Create account")').click();

    // Verify on register page
    await expect(page.locator('input[type="email"]')).toBeVisible();
    await expect(page.locator('h2')).toContainText('Register');
  });
});
```

### 2. Location Management Tests

```typescript
// e2e/locations.spec.ts
import { test, expect, Page } from '@playwright/test';

test.describe('Location Management', () => {
  let page: Page;

  test.beforeEach(async ({ page: testPage }) => {
    page = testPage;
    // Login first
    await loginAs(page, 'test@example.com', 'Test123!');
    // Navigate to locations page
    await page.goto('/#locations');
  });

  test('should display locations list', async () => {
    // Wait for table to load
    await page.waitForSelector('table tbody');

    // Check locations are rendered
    const rows = await page.locator('table tbody tr').count();
    expect(rows).toBeGreaterThan(0);
  });

  test('should create new location', async () => {
    // Get initial count
    const initialCount = await page.locator('table tbody tr').count();

    // Fill form
    await page.locator('input[placeholder="Location name"]').fill('New City');
    await page.locator('input[placeholder="Latitude"]').fill('40.7128');
    await page.locator('input[placeholder="Longitude"]').fill('-74.0060');

    // Submit
    await page.locator('button:has-text("Add Location")').click();

    // Check success toast
    await expect(
      page.locator('text=Location Provisioned')
    ).toBeVisible();

    // Verify list updated
    const newCount = await page.locator('table tbody tr').count();
    expect(newCount).toBe(initialCount + 1);
  });

  test('should edit location', async () => {
    // Find first location edit button
    const editButton = page.locator('button:has-text("Edit")').first();
    await editButton.click();

    // Edit modal appears
    const nameInput = page.locator('input[placeholder="Location name"]');
    await nameInput.fill('Updated City');

    // Save
    await page.locator('button:has-text("Update")').click();

    // Verify success
    await expect(
      page.locator('text=Coordinates Corrected')
    ).toBeVisible();
    await expect(page.locator('text=Updated City')).toBeVisible();
  });

  test('should delete location', async () => {
    const initialCount = await page.locator('table tbody tr').count();

    // Find and click delete button
    const deleteButton = page.locator('button:has-text("Delete")').first();
    await deleteButton.click();

    // Confirm deletion
    await page.locator('button:has-text("Confirm")').click();

    // Check success toast
    await expect(
      page.locator('text=Decommissioned Station')
    ).toBeVisible();

    // Verify count decreased
    const newCount = await page.locator('table tbody tr').count();
    expect(newCount).toBe(initialCount - 1);
  });
});

// Helper function
async function loginAs(
  page: Page,
  email: string,
  password: string
) {
  await page.goto('/');
  await page.locator('input[type="email"]').fill(email);
  await page.locator('input[type="password"]').fill(password);
  await page.locator('button:has-text("Login")').click();
  await page.waitForURL('**/overview');
}
```

### 3. Rules Management Tests

```typescript
// e2e/rules.spec.ts
import { test, expect, Page } from '@playwright/test';

test.describe('Rules Management', () => {
  let page: Page;

  test.beforeEach(async ({ page: testPage }) => {
    page = testPage;
    await loginAs(page, 'test@example.com', 'Test123!');
    await page.goto('/#rules');
  });

  test('should create weather rule', async () => {
    const initialCount = await page
      .locator('[data-testid="rule-card"]')
      .count();

    // Select location
    await page.locator('select[name="location"]').selectOption('loc-1');

    // Select metric
    await page.locator('select[name="metric"]').selectOption('temperature');

    // Select operator
    await page.locator('select[name="operator"]').selectOption('>');

    // Enter threshold
    await page.locator('input[name="threshold"]').fill('30');

    // Create rule
    await page.locator('button:has-text("Create Rule")').click();

    // Verify success
    await expect(page.locator('text=Alert Rule Armed')).toBeVisible();

    const newCount = await page
      .locator('[data-testid="rule-card"]')
      .count();
    expect(newCount).toBe(initialCount + 1);
  });

  test('should toggle rule active status', async () => {
    const toggleButton = page
      .locator('[data-testid="rule-toggle"]')
      .first();

    // Get initial state
    const initialState = await toggleButton.getAttribute('data-active');

    // Click toggle
    await toggleButton.click();

    // Wait for toast
    await page.waitForSelector('text=Incident Trigger');

    // Verify state changed
    const newState = await toggleButton.getAttribute('data-active');
    expect(newState).not.toBe(initialState);
  });
});
```

### 4. Alerts Management Tests

```typescript
// e2e/alerts.spec.ts
import { test, expect, Page } from '@playwright/test';

test.describe('Alerts Management', () => {
  let page: Page;

  test.beforeEach(async ({ page: testPage }) => {
    page = testPage;
    await loginAs(page, 'test@example.com', 'Test123!');
    await page.goto('/#alerts');
  });

  test('should display active alerts', async () => {
    // Wait for alerts to load
    await page.waitForSelector('[data-testid="alert-card"]');

    // Check alerts are visible
    const alerts = page.locator('[data-testid="alert-card"]');
    const count = await alerts.count();

    expect(count).toBeGreaterThan(0);
  });

  test('should resolve alert', async () => {
    const initialCount = await page
      .locator('[data-testid="alert-card"]')
      .count();

    // Click resolve on first alert
    const resolveButton = page
      .locator('button:has-text("Resolve")')
      .first();
    await resolveButton.click();

    // Verify success toast
    await expect(
      page.locator('text=Incident Acknowledged')
    ).toBeVisible();

    // Wait for list update
    await page.waitForTimeout(500);

    // Verify count decreased
    const newCount = await page
      .locator('[data-testid="alert-card"]')
      .count();
    expect(newCount).toBeLessThan(initialCount);
  });

  test('should filter alerts by severity', async () => {
    // Click HIGH severity filter
    await page.locator('input[value="HIGH"]').check();

    // Wait for filtered results
    await page.waitForTimeout(500);

    // Verify only HIGH severity alerts shown
    const alerts = page.locator('[data-testid="alert-card"]');
    const count = await alerts.count();

    // Check badge shows HIGH
    for (let i = 0; i < count; i++) {
      const badge = alerts.nth(i).locator('[data-severity]');
      const severity = await badge.getAttribute('data-severity');
      expect(severity).toBe('HIGH');
    }
  });
});
```

---

## Best Practices for E2E Tests

### 1. Test Selectors

```typescript
// Good - use data attributes
await page.locator('[data-testid="submit-button"]').click();

// Good - specific text
await page.locator('button:has-text("Create")').click();

// Acceptable - role
await page.locator('role=button[name="Create"]').click();

// Bad - brittle selectors
await page.locator('.btn-primary:nth-child(3)').click();
await page.locator('button.action > span').click();
```

### 2. Wait Strategies

```typescript
// Good - wait for specific element
await page.waitForSelector('[data-testid="alert-card"]');

// Good - wait for URL
await page.waitForURL('**/locations');

// Good - wait for element to be visible
await expect(page.locator('text=Success')).toBeVisible();

// Bad - arbitrary wait
await page.waitForTimeout(2000);
```

### 3. Test Organization

```typescript
test.describe('Feature Name', () => {
  let page: Page;

  test.beforeEach(async ({ page: testPage }) => {
    // Setup before each test
    page = testPage;
    await setupTest(page);
  });

  test.afterEach(async () => {
    // Cleanup after each test
    await teardownTest();
  });

  test('should do something', async () => {
    // AAA pattern: Arrange, Act, Assert
    // Arrange - set up test data
    // Act - perform action
    // Assert - verify result
  });
});
```

### 4. Error Handling

```typescript
// Good - handle expected errors
test('should show error on invalid input', async ({ page }) => {
  await page.locator('input').fill('');
  await page.locator('button').click();

  // Expect error message
  await expect(page.locator('text=Required field')).toBeVisible();
});

// Good - handle network errors
test('should handle API failure gracefully', async ({ page }) => {
  // Mock API failure
  await page.route('**/api/locations', (route) => {
    route.abort('failed');
  });

  await page.goto('/#locations');

  // Expect error message
  await expect(page.locator('text=Failed to load')).toBeVisible();
});
```

---

## Unit Testing for Utilities

While the project primarily uses E2E tests, here's how to test utility functions:

```typescript
// src/lib/validation.ts
export function validateEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

export function validateCoordinates(
  lat: number,
  lng: number
): boolean {
  return lat >= -90 && lat <= 90 && lng >= -180 && lng <= 180;
}

// src/lib/validation.test.ts
import { describe, it, expect } from 'vitest';
import { validateEmail, validateCoordinates } from './validation';

describe('Email Validation', () => {
  it('should accept valid email', () => {
    expect(validateEmail('test@example.com')).toBe(true);
  });

  it('should reject invalid email', () => {
    expect(validateEmail('invalid-email')).toBe(false);
    expect(validateEmail('test@')).toBe(false);
  });
});

describe('Coordinate Validation', () => {
  it('should accept valid coordinates', () => {
    expect(validateCoordinates(40.7128, -74.006)).toBe(true);
    expect(validateCoordinates(0, 0)).toBe(true);
  });

  it('should reject invalid coordinates', () => {
    expect(validateCoordinates(91, 0)).toBe(false); // lat too high
    expect(validateCoordinates(0, 181)).toBe(false); // lng too high
  });
});
```

---

## Debugging Tests

### 1. Playwright Inspector

```bash
# Open Playwright Inspector
npx playwright test --debug

# Step through test execution
# Use Inspector UI to pause and inspect
```

### 2. Headed Mode

```bash
# Run tests with visible browser
npm run test:e2e:headed

# Watch browser interactions in real-time
```

### 3. Screenshots & Videos

```typescript
// Automatic on failure
export default {
  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
};

// View screenshots
// test-results/*/screenshots
```

### 4. Test Reports

```bash
# Generate HTML report
npm run test:e2e -- --reporter=html

# View report
npx playwright show-report
```

---

## CI/CD Integration

### GitHub Actions Example

```yaml
# .github/workflows/test.yml
name: E2E Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Install Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Start dev server
        run: npm run dev &

      - name: Wait for server
        run: sleep 5

      - name: Run tests
        run: npm run test:e2e

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Test Coverage Goals

```
Target Coverage:
├─ Statements: 70%+
├─ Branches: 65%+
├─ Functions: 70%+
└─ Lines: 70%+

Critical Paths (Must Test):
├─ Authentication
├─ Location CRUD
├─ Rule Creation & Management
├─ Alert Resolution
└─ Data Export
```

---

## Test Naming Convention

```typescript
// Good test names
test('should display login form when user is not authenticated', async () => {});
test('should login successfully with valid credentials', async () => {});
test('should show error message on invalid credentials', async () => {});
test('should create new location with valid data', async () => {});
test('should prevent location creation without required fields', async () => {});
```

---

## Performance Testing

```bash
# Monitor API response times
# Use Playwright's built-in network inspection
test('should load locations within 2 seconds', async ({ page }) => {
  const startTime = Date.now();
  
  await page.goto('/#locations');
  await page.waitForSelector('table tbody');
  
  const loadTime = Date.now() - startTime;
  expect(loadTime).toBeLessThan(2000);
});
```

---

**Last Updated**: June 2026  
**Version**: 1.0.0

**Next Steps**: Read [DEPLOYMENT.md](./DEPLOYMENT.md) for deployment procedures.
