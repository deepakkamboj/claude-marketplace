# Playwright Accessibility Testing Integration

This guide covers how to integrate comprehensive accessibility testing into your project using Playwright and axe-core. The accessibility plugin provides automated test generation and best practices for WCAG 2.1 Level AA compliance verification.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Test Structure](#test-structure)
- [Using the generate-a11y-tests Skill](#using-the-generate-a11y-tests-skill)
- [Running Tests](#running-tests)
- [CI/CD Integration](#cicd-integration)
- [Debugging Violations](#debugging-violations)
- [Common Test Patterns](#common-test-patterns)
- [Framework-Specific Examples](#framework-specific-examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Installation

### Prerequisites

- Node.js 16+ installed
- Existing project with UI components or pages to test

### Install Dependencies

```bash
# Using npm
npm install -D @playwright/test @axe-core/playwright

# Using yarn
yarn add -D @playwright/test @axe-core/playwright

# Using pnpm
pnpm add -D @playwright/test @axe-core/playwright
```

### Initialize Playwright (if not already installed)

```bash
npx playwright install
```

This installs the required browsers (Chromium, Firefox, WebKit).

## Quick Start

### 1. Generate Your First Accessibility Test

Using Claude Code with the accessibility plugin:

```
/accessibility:generate-a11y-tests

User: "Generate accessibility tests for src/components/Button.tsx"
```

Claude will analyze your component and create a comprehensive test file at `tests/accessibility/button.spec.ts`.

### 2. Run the Test

```bash
npx playwright test tests/accessibility/button.spec.ts
```

### 3. View Results

```bash
# Generate HTML report
npx playwright show-report
```

## Configuration

### Basic playwright.config.ts

Create or update `playwright.config.ts` in your project root:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  // Run tests in files in parallel
  fullyParallel: true,

  // Fail the build on CI if you accidentally left test.only
  forbidOnly: !!process.env.CI,

  // Retry on CI only
  retries: process.env.CI ? 2 : 0,

  // Opt out of parallel tests on CI for accessibility consistency
  workers: process.env.CI ? 1 : undefined,

  // Reporter to use
  reporter: [
    ['html'],
    ['list'],
    ['json', { outputFile: 'test-results/results.json' }],
  ],

  use: {
    // Base URL for navigation
    baseURL: 'http://localhost:3000',

    // Collect trace when retrying the failed test
    trace: 'on-first-retry',

    // Screenshots on failure
    screenshot: 'only-on-failure',

    // Slower navigation for accessibility tools
    navigationTimeout: 30000,
  },

  // Configure projects for major browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },

    // Separate accessibility project
    {
      name: 'accessibility',
      testDir: './tests/accessibility',
      use: {
        ...devices['Desktop Chrome'],
        // Disable parallel for consistency
        workers: 1,
      },
    },
  ],

  // Run your local dev server before starting the tests
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Accessibility-Specific Configuration

For dedicated accessibility testing, create `tests/accessibility/playwright.config.ts`:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './',

  // Accessibility tests should run sequentially for consistency
  fullyParallel: false,
  workers: 1,

  timeout: 60000, // Longer timeout for accessibility scans

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',

    // Slow down for accessibility tools
    navigationTimeout: 30000,
    actionTimeout: 15000,
  },

  projects: [
    {
      name: 'a11y-chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'a11y-firefox',
      use: { ...devices['Desktop Firefox'] },
    },
  ],

  reporter: [
    ['html', { outputFolder: 'playwright-report/accessibility' }],
    ['json', { outputFile: 'test-results/accessibility-results.json' }],
  ],
});
```

## Test Structure

### Basic Test File Structure

All accessibility tests follow this structure:

```typescript
import { test, expect } from '@playwright/test';
import { AxeBuilder } from '@axe-core/playwright';

test.describe('Component/Page Accessibility', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to component/page
    await page.goto('/component-url');
  });

  test('has no WCAG violations', async ({ page }) => {
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  // Additional specific tests...
});
```

### Test File Organization

Recommended directory structure:

```
tests/
├── accessibility/
│   ├── components/
│   │   ├── button.spec.ts
│   │   ├── form.spec.ts
│   │   ├── modal.spec.ts
│   │   └── navigation.spec.ts
│   ├── pages/
│   │   ├── home-page.spec.ts
│   │   ├── dashboard-page.spec.ts
│   │   └── checkout-page.spec.ts
│   └── workflows/
│       ├── user-registration.spec.ts
│       └── purchase-flow.spec.ts
└── playwright.config.ts
```

## Using the generate-a11y-tests Skill

The `generate-a11y-tests` skill automates creation of comprehensive accessibility tests.

### Basic Usage

```
/accessibility:generate-a11y-tests

User: "Generate accessibility tests for src/components/LoginForm.tsx"
```

Claude will:
1. Analyze your component structure
2. Identify interactive elements and ARIA usage
3. Generate comprehensive tests including:
   - axe-core automated scanning
   - Keyboard navigation tests
   - Focus management tests
   - ARIA attribute verification
   - Form accessibility checks (if applicable)

### Advanced Usage

Request specific test scenarios:

```
User: "Generate accessibility tests for src/components/Modal.tsx focusing on focus trapping and keyboard shortcuts"
```

```
User: "Create page-level accessibility tests for /checkout with full keyboard navigation flow"
```

### Generated Test Components

Every generated test includes:

1. **Automated axe-core scanning** - WCAG 2.0/2.1 Level A and AA
2. **Keyboard operability tests** - Tab navigation, keyboard shortcuts
3. **Focus management tests** - Focus indicators, focus trapping (modals)
4. **ARIA verification** - Roles, states, properties
5. **Component-specific tests** - Based on component type (form, modal, etc.)

## Running Tests

### Run All Accessibility Tests

```bash
npx playwright test tests/accessibility
```

### Run Specific Test File

```bash
npx playwright test tests/accessibility/button.spec.ts
```

### Run with UI Mode (Interactive)

```bash
npx playwright test tests/accessibility --ui
```

### Run in Headed Mode (See Browser)

```bash
npx playwright test tests/accessibility --headed
```

### Run in Debug Mode

```bash
npx playwright test tests/accessibility --debug
```

### Run Specific Project

```bash
npx playwright test --project=accessibility
```

### Generate HTML Report

```bash
npx playwright test tests/accessibility --reporter=html
npx playwright show-report
```

### Watch Mode (Re-run on Changes)

```bash
npx playwright test tests/accessibility --watch
```

## CI/CD Integration

### GitHub Actions

Create `.github/workflows/accessibility-tests.yml`:

```yaml
name: Accessibility Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  accessibility:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Build application
        run: npm run build

      - name: Run accessibility tests
        run: npx playwright test tests/accessibility

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: accessibility-test-results
          path: |
            test-results/
            playwright-report/

      - name: Comment PR with results
        if: github.event_name == 'pull_request' && failure()
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '❌ Accessibility tests failed. Please review the test results.'
            })
```

### GitLab CI

Create or update `.gitlab-ci.yml`:

```yaml
accessibility-tests:
  stage: test
  image: mcr.microsoft.com/playwright:v1.40.0-focal
  script:
    - npm ci
    - npx playwright install chromium
    - npm run build
    - npx playwright test tests/accessibility
  artifacts:
    when: always
    paths:
      - test-results/
      - playwright-report/
    reports:
      junit: test-results/results.xml
```

### Jenkins

```groovy
pipeline {
  agent any

  stages {
    stage('Install Dependencies') {
      steps {
        sh 'npm ci'
        sh 'npx playwright install --with-deps'
      }
    }

    stage('Build') {
      steps {
        sh 'npm run build'
      }
    }

    stage('Accessibility Tests') {
      steps {
        sh 'npx playwright test tests/accessibility'
      }
    }
  }

  post {
    always {
      publishHTML([
        allowMissing: false,
        alwaysLinkToLastBuild: true,
        keepAll: true,
        reportDir: 'playwright-report',
        reportFiles: 'index.html',
        reportName: 'Accessibility Test Report'
      ])
    }
  }
}
```

### CircleCI

Create `.circleci/config.yml`:

```yaml
version: 2.1

jobs:
  accessibility-tests:
    docker:
      - image: mcr.microsoft.com/playwright:v1.40.0-focal
    steps:
      - checkout
      - restore_cache:
          keys:
            - npm-deps-{{ checksum "package-lock.json" }}
      - run: npm ci
      - save_cache:
          key: npm-deps-{{ checksum "package-lock.json" }}
          paths:
            - node_modules
      - run: npx playwright install chromium
      - run: npm run build
      - run: npx playwright test tests/accessibility
      - store_artifacts:
          path: playwright-report
      - store_test_results:
          path: test-results

workflows:
  test:
    jobs:
      - accessibility-tests
```

## Debugging Violations

### Understanding axe-core Violations

When tests fail, axe-core provides detailed violation information:

```typescript
{
  id: 'color-contrast',
  impact: 'serious',
  description: 'Ensures the contrast between foreground and background colors meets WCAG 2 AA contrast ratio thresholds',
  help: 'Elements must have sufficient color contrast',
  helpUrl: 'https://dequeuniversity.com/rules/axe/4.8/color-contrast',
  nodes: [
    {
      html: '<button class="btn-primary">Submit</button>',
      target: ['button.btn-primary'],
      failureSummary: 'Element has insufficient color contrast of 3.2:1 (foreground color: #5061ff, background color: #ffffff, font size: 12.0pt, font weight: normal). Expected contrast ratio of 4.5:1'
    }
  ]
}
```

### Debugging Steps

1. **Run test with --debug flag**:
   ```bash
   npx playwright test tests/accessibility/button.spec.ts --debug
   ```

2. **Use headed mode to see violations**:
   ```bash
   npx playwright test tests/accessibility/button.spec.ts --headed --headed-mode=debug
   ```

3. **Inspect specific violations**:
   ```typescript
   test('check specific element', async ({ page }) => {
     const results = await new AxeBuilder({ page })
       .include('.specific-component')
       .analyze();

     console.log('Violations:', JSON.stringify(results.violations, null, 2));
   });
   ```

4. **Use browser DevTools**:
   ```typescript
   test('debug with pause', async ({ page }) => {
     await page.goto('/component');
     await page.pause(); // Opens Playwright Inspector
   });
   ```

### Filtering Violations

Focus on specific violation types:

```typescript
test('check only color contrast', async ({ page }) => {
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2aa'])
    .include('.main-content')
    .exclude('.legacy-widget') // Exclude known issues
    .options({
      rules: {
        'color-contrast': { enabled: true },
      },
    })
    .analyze();

  expect(results.violations).toEqual([]);
});
```

### Excluding Known Issues (Temporarily)

While fixing violations, you can temporarily exclude elements:

```typescript
test('accessibility scan with exclusions', async ({ page }) => {
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .exclude('#third-party-widget') // TODO: Fix in next sprint
    .analyze();

  expect(results.violations).toEqual([]);
});
```

**Important**: Document exclusions and create tickets to fix them.

## Common Test Patterns

### Pattern 1: Form Accessibility

```typescript
test.describe('Form Accessibility', () => {
  test('all inputs have labels', async ({ page }) => {
    await page.goto('/form');

    const inputs = await page.locator('input, textarea, select');
    const count = await inputs.count();

    for (let i = 0; i < count; i++) {
      const input = inputs.nth(i);
      const inputId = await input.getAttribute('id');
      const inputName = await input.getAttribute('name');

      // Check for associated label
      const label = await page.locator(`label[for="${inputId}"]`);
      const labelExists = (await label.count()) > 0;

      // Or wrapped in label
      const wrappedInLabel = await input.evaluate(el => {
        return el.closest('label') !== null;
      });

      // Or has aria-label
      const ariaLabel = await input.getAttribute('aria-label');

      expect(labelExists || wrappedInLabel || ariaLabel).toBeTruthy();
    }
  });

  test('error messages are accessible', async ({ page }) => {
    await page.goto('/form');

    // Submit with invalid data
    await page.locator('button[type="submit"]').click();

    // Check for accessible error messages
    const errorMessages = await page.locator('[role="alert"], [aria-live]');
    expect(await errorMessages.count()).toBeGreaterThan(0);

    // Check errors are associated with inputs
    const invalidInputs = await page.locator('[aria-invalid="true"]');
    const invalidCount = await invalidInputs.count();

    for (let i = 0; i < invalidCount; i++) {
      const input = invalidInputs.nth(i);
      const describedBy = await input.getAttribute('aria-describedby');
      expect(describedBy).toBeTruthy();
    }
  });
});
```

### Pattern 2: Modal Focus Management

```typescript
test.describe('Modal Focus Management', () => {
  test('traps focus within modal', async ({ page }) => {
    await page.goto('/page-with-modal');

    const trigger = await page.locator('button[data-modal-trigger]');
    await trigger.click();

    const modal = await page.locator('[role="dialog"]');
    await modal.waitFor();

    // Get focusable elements in modal
    const focusableInModal = await modal.locator(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const modalElementCount = await focusableInModal.count();

    // Tab through more times than there are elements
    for (let i = 0; i < modalElementCount + 3; i++) {
      await page.keyboard.press('Tab');

      // Verify focus is still in modal
      const focused = await page.locator(':focus');
      const isInModal = await focused.evaluate(
        (el, modalEl) => modalEl.contains(el),
        await modal.elementHandle()
      );

      expect(isInModal).toBe(true);
    }
  });

  test('restores focus on close', async ({ page }) => {
    await page.goto('/page-with-modal');

    const trigger = await page.locator('button[data-modal-trigger]');
    await trigger.click();

    const modal = await page.locator('[role="dialog"]');
    await modal.waitFor();

    // Close modal
    await page.keyboard.press('Escape');

    await expect(modal).not.toBeVisible();

    // Verify focus restored
    const focused = await page.locator(':focus');
    const isTrigger = await focused.evaluate(
      (el, triggerEl) => el === triggerEl,
      await trigger.elementHandle()
    );

    expect(isTrigger).toBe(true);
  });
});
```

### Pattern 3: Keyboard Navigation

```typescript
test.describe('Keyboard Navigation', () => {
  test('all interactive elements are keyboard accessible', async ({ page }) => {
    await page.goto('/page');

    // Get all potentially interactive elements
    const interactive = await page.locator(
      'a, button, input, select, textarea, [role="button"], [role="link"], [tabindex]:not([tabindex="-1"])'
    );
    const count = await interactive.count();

    for (let i = 0; i < count; i++) {
      const element = interactive.nth(i);

      // Focus element
      await element.focus();

      // Verify it's focused
      const isFocused = await element.evaluate(el => el === document.activeElement);
      expect(isFocused).toBe(true);

      // Verify visible focus indicator
      const hasFocusIndicator = await element.evaluate(el => {
        const styles = window.getComputedStyle(el);
        return (
          styles.outline !== 'none' ||
          styles.outlineWidth !== '0px' ||
          styles.boxShadow !== 'none'
        );
      });

      expect(hasFocusIndicator).toBe(true);
    }
  });

  test('skip link works', async ({ page }) => {
    await page.goto('/');

    // Tab to skip link (usually first focusable)
    await page.keyboard.press('Tab');

    const focused = await page.locator(':focus');
    const text = await focused.textContent();

    expect(text?.toLowerCase()).toMatch(/skip/);

    // Activate skip link
    await page.keyboard.press('Enter');

    // Verify jumped to main content
    const newFocus = await page.locator(':focus');
    const newFocusId = await newFocus.getAttribute('id');

    expect(newFocusId).toMatch(/main|content/);
  });
});
```

### Pattern 4: Dynamic Content Announcements

```typescript
test.describe('Dynamic Content Announcements', () => {
  test('loading states are announced', async ({ page }) => {
    await page.goto('/dashboard');

    // Check for loading announcement
    const loadingRegion = await page.locator('[aria-live="polite"], [aria-live="assertive"]');
    const ariaLive = await loadingRegion.getAttribute('aria-live');

    expect(['polite', 'assertive']).toContain(ariaLive);
  });

  test('success messages are announced', async ({ page }) => {
    await page.goto('/form');

    // Submit form
    await page.fill('input[name="email"]', 'test@example.com');
    await page.click('button[type="submit"]');

    // Wait for success message
    const successMessage = await page.locator('[role="status"], [role="alert"]');
    await expect(successMessage).toBeVisible();

    // Verify it has aria-live
    const ariaLive = await successMessage.getAttribute('aria-live');
    expect(ariaLive).toBeTruthy();
  });
});
```

## Framework-Specific Examples

### React

```typescript
// Component: src/components/Button.tsx
import { test, expect } from '@playwright/test';
import { AxeBuilder } from '@axe-core/playwright';

test.describe('React Button Component', () => {
  test.beforeEach(async ({ page }) => {
    // Assuming you have a dev server running or Storybook
    await page.goto('http://localhost:6006/?path=/story/button--primary');
  });

  test('has no accessibility violations', async ({ page }) => {
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('handles keyboard interactions', async ({ page }) => {
    const button = await page.locator('button').first();

    await button.focus();
    await page.keyboard.press('Enter');

    // Verify button action (adjust based on your component)
    const clickCount = await page.textContent('[data-testid="click-count"]');
    expect(clickCount).toBe('1');
  });
});
```

### Vue

```typescript
// Component: src/components/Modal.vue
import { test, expect } from '@playwright/test';
import { AxeBuilder } from '@axe-core/playwright';

test.describe('Vue Modal Component', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173/modal-demo');
  });

  test('modal has correct ARIA attributes', async ({ page }) => {
    await page.click('[data-testid="open-modal"]');

    const modal = await page.locator('[role="dialog"]');
    await expect(modal).toBeVisible();
    await expect(modal).toHaveAttribute('aria-modal', 'true');
    await expect(modal).toHaveAttribute('aria-labelledby');
  });

  test('v-focus directive works', async ({ page }) => {
    await page.click('[data-testid="open-modal"]');

    const modal = await page.locator('[role="dialog"]');
    await modal.waitFor();

    // Verify focus is inside modal
    const focused = await page.locator(':focus');
    const isInModal = await focused.evaluate(
      (el, modalEl) => modalEl.contains(el),
      await modal.elementHandle()
    );

    expect(isInModal).toBe(true);
  });
});
```

### Angular

```typescript
// Component: src/app/components/dropdown/dropdown.component.ts
import { test, expect } from '@playwright/test';
import { AxeBuilder } from '@axe-core/playwright';

test.describe('Angular Dropdown Component', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:4200/components/dropdown');
  });

  test('dropdown has no violations', async ({ page }) => {
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('supports arrow key navigation', async ({ page }) => {
    const trigger = await page.locator('[aria-haspopup="listbox"]');
    await trigger.click();

    await page.keyboard.press('ArrowDown');
    let focused = await page.locator(':focus');
    await expect(focused).toHaveAttribute('role', 'option');

    await page.keyboard.press('ArrowDown');
    focused = await page.locator(':focus');
    await expect(focused).toHaveAttribute('role', 'option');
  });
});
```

### Svelte

```typescript
// Component: src/lib/Accordion.svelte
import { test, expect } from '@playwright/test';
import { AxeBuilder } from '@axe-core/playwright';

test.describe('Svelte Accordion Component', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173/accordion');
  });

  test('accordion has correct ARIA', async ({ page }) => {
    const buttons = await page.locator('[aria-expanded]');
    const count = await buttons.count();

    for (let i = 0; i < count; i++) {
      const button = buttons.nth(i);
      const controls = await button.getAttribute('aria-controls');
      const expanded = await button.getAttribute('aria-expanded');

      expect(controls).toBeTruthy();
      expect(['true', 'false']).toContain(expanded);
    }
  });
});
```

## Best Practices

### 1. Test at Multiple Levels

- **Component level**: Individual UI components in isolation
- **Page level**: Complete pages with full context
- **Workflow level**: Multi-step user journeys

### 2. Combine Automated and Manual Tests

```typescript
test.describe('Comprehensive Accessibility', () => {
  // Automated scanning
  test('automated axe-core scan', async ({ page }) => {
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();
    expect(results.violations).toEqual([]);
  });

  // Manual verification
  test('keyboard navigation works', async ({ page }) => {
    // Explicit keyboard interaction tests
  });

  test('screen reader announcements', async ({ page }) => {
    // Verify ARIA live regions and announcements
  });
});
```

### 3. Test Different States

```typescript
test.describe('Button States', () => {
  test('default state has no violations', async ({ page }) => {
    // Test default state
  });

  test('disabled state is accessible', async ({ page }) => {
    const button = await page.locator('button[disabled]');
    await expect(button).toHaveAttribute('disabled');
    await expect(button).toHaveAttribute('aria-disabled', 'true');
  });

  test('loading state is announced', async ({ page }) => {
    // Trigger loading state
    await page.click('button[data-async]');

    const button = await page.locator('button[data-async]');
    const ariaLabel = await button.getAttribute('aria-label');
    expect(ariaLabel).toMatch(/loading/i);
  });
});
```

### 4. Use Descriptive Test Names

```typescript
// ✅ Good
test('form error messages are associated with inputs via aria-describedby', async ({ page }) => {
  // ...
});

// ❌ Bad
test('test form errors', async ({ page }) => {
  // ...
});
```

### 5. Test Real User Flows

```typescript
test.describe('Registration Flow Accessibility', () => {
  test('user can complete registration with keyboard only', async ({ page }) => {
    await page.goto('/register');

    // Fill form using keyboard
    await page.keyboard.press('Tab'); // Focus first input
    await page.keyboard.type('John Doe');

    await page.keyboard.press('Tab');
    await page.keyboard.type('john@example.com');

    await page.keyboard.press('Tab');
    await page.keyboard.type('password123');

    await page.keyboard.press('Tab');
    await page.keyboard.type('password123');

    // Submit with keyboard
    await page.keyboard.press('Enter');

    // Verify success
    await expect(page).toHaveURL(/.*dashboard/);
  });
});
```

### 6. Document Known Issues

```typescript
test('navigation has known contrast issues', async ({ page }) => {
  // TODO: Fix contrast on navigation links (Ticket #1234)
  // Expected: 4.5:1, Actual: 3.8:1
  // Exclude for now, must be fixed before v2.0 release

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .exclude('nav.main-nav') // Known issue
    .analyze();

  expect(results.violations).toEqual([]);
});
```

### 7. Regular Test Maintenance

- **Review and update tests** when components change
- **Remove exclusions** as violations are fixed
- **Add tests** for new components immediately
- **Run tests** before every commit and in CI/CD

## Troubleshooting

### Issue: Tests timeout

**Solution**: Increase timeout in configuration

```typescript
test('slow component', async ({ page }) => {
  test.setTimeout(60000); // 60 seconds

  await page.goto('/complex-page');
  // ...
});
```

### Issue: Flaky tests

**Solution**: Use proper waits

```typescript
// ❌ Bad - timing-based
await page.waitForTimeout(1000);

// ✅ Good - condition-based
await page.waitForSelector('[role="dialog"]');
await page.locator('[role="dialog"]').waitFor({ state: 'visible' });
```

### Issue: False positives from third-party widgets

**Solution**: Exclude third-party code

```typescript
const results = await new AxeBuilder({ page })
  .exclude('#third-party-chat-widget')
  .exclude('.google-maps-embed')
  .analyze();
```

### Issue: Different results locally vs CI

**Solution**: Use consistent browser and viewport

```typescript
// playwright.config.ts
use: {
  viewport: { width: 1280, height: 720 },
  deviceScaleFactor: 1,
},
```

### Issue: Can't test components in isolation

**Solution**: Use Storybook or similar

```bash
npm install -D @storybook/addon-a11y

# Run Storybook
npm run storybook

# Test in Playwright
await page.goto('http://localhost:6006/?path=/story/button--primary');
```

## Additional Resources

- [Playwright Documentation](https://playwright.dev/)
- [axe-core Rules](https://github.com/dequelabs/axe-core/blob/develop/doc/rule-descriptions.md)
- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [Accessibility Testing Guide](https://www.w3.org/WAI/test-evaluate/)
- [ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)

## Getting Help

- Use the `accessibility:tester` agent for comprehensive audits
- Invoke `accessibility:contrast-checker` for color issues
- Reference `ACCESSIBILITY_STANDARDS.md` for WCAG criteria
- Check Playwright Discord for technical questions

## Summary

This integration enables:
- ✅ Automated WCAG 2.1 Level AA testing
- ✅ Keyboard accessibility verification
- ✅ ARIA implementation validation
- ✅ Focus management testing
- ✅ Form accessibility checks
- ✅ CI/CD integration
- ✅ Comprehensive reporting

Start with automated axe-core scans, then add manual keyboard and interaction tests for complete accessibility coverage.
