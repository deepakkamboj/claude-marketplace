# Accessibility Plugin - Developer Usage Guide

Complete guide for using the accessibility plugin throughout all stages of development, from planning to deployment.

## Table of Contents

- [Installation](#installation)
- [Quick Reference](#quick-reference)
- [Development Lifecycle Stages](#development-lifecycle-stages)
  - [Stage 1: Project Setup & Planning](#stage-1-project-setup--planning)
  - [Stage 2: Active Development](#stage-2-active-development)
  - [Stage 3: Component/Feature Review](#stage-3-componentfeature-review)
  - [Stage 4: Testing & Validation](#stage-4-testing--validation)
  - [Stage 5: Bug Fixing & Refactoring](#stage-5-bug-fixing--refactoring)
  - [Stage 6: Pre-Commit & Code Review](#stage-6-pre-commit--code-review)
  - [Stage 7: CI/CD & Deployment](#stage-7-cicd--deployment)
  - [Stage 8: Maintenance & Audits](#stage-8-maintenance--audits)
- [Common Scenarios](#common-scenarios)
- [Skill & Agent Reference](#skill--agent-reference)
- [Troubleshooting](#troubleshooting)

---

## Installation

### Option 1: Via Claude Code Marketplace (Recommended)

```bash
# Add DevBuild Studio marketplace
claude plugin marketplace add @deepakkamboj/marketplace

# Install the accessibility plugin
claude plugin install accessibility@deepakkamboj
```

### Option 2: Manual Installation via Config

Add to `~/.claude/config.json` (global) or `.claude/config.json` (project-specific):

```json
{
  "plugins": [
    {
      "name": "accessibility",
      "source": {
        "source": "github",
        "repo": "deepakkamboj/accessibility",
        "path": "plugins/accessibility"
      }
    }
  ]
}
```

Then run:
```bash
claude plugin sync
```

### Verify Installation

```bash
# List installed plugins
claude plugin list

# Should show: accessibility v1.0.0
```

---

## Quick Reference

| When | Command | What It Does |
|------|---------|--------------|
| Starting new feature | `Skill({ skill: "accessibility:accessible-dev" })` | Enables accessible-first development mode |
| **Quick runtime scan** | `Task({ subagent_type: "tester", prompt: "Scan http://localhost:3000/page" })` | **Instant axe-core scan of live URL** |
| Reviewing existing code | `Task({ subagent_type: "tester", prompt: "Test [path] for accessibility" })` | Comprehensive accessibility audit |
| Checking colors only | `Skill({ skill: "accessibility:contrast-checker" })` | Analyzes color contrast ratios |
| Checking color indicators | `Skill({ skill: "accessibility:use-of-color" })` | Finds color-only information |
| Checking link text | `Skill({ skill: "accessibility:link-purpose" })` | Validates link descriptions |
| Generating tests | `Skill({ skill: "accessibility:generate-a11y-tests" })` | Creates Playwright accessibility tests |
| Fixing violations | `Skill({ skill: "accessibility:refactor" })` | Automatically fixes accessibility issues |
| **Batch scan multiple pages** | `Task({ subagent_type: "tester", prompt: "Scan [URL1, URL2, URL3]" })` | **Scan multiple URLs at once** |

---

## Development Lifecycle Stages

### Stage 1: Project Setup & Planning

**Goal:** Establish accessibility requirements and standards before writing code.

#### Commands & Prompts

**1.1 Review Project Accessibility Standards**

```
Prompt: "Read ACCESSIBILITY_STANDARDS.md and summarize the key WCAG 2.1 AA requirements we need to follow for this project."
```

**Expected Output:** Summary of applicable standards, critical requirements, and project-specific rules.

---

**1.2 Audit Existing Codebase (for existing projects)**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Perform a comprehensive accessibility audit of the entire codebase. Focus on src/components/ and src/pages/. Generate a summary report prioritizing Critical and High severity issues."
})
```

**Expected Output:**
- List of all accessibility violations by severity
- Affected files and line numbers
- Recommended fixes
- Estimated effort to remediate

---

**1.3 Establish Testing Strategy**

```
Prompt: "Based on the accessibility standards, what Playwright accessibility tests should we have for this project? List the critical test scenarios."
```

**Expected Output:** Test plan covering forms, navigation, modals, interactive components, etc.

---

**1.4 Setup Pre-Commit Hooks (Optional)**

```
Prompt: "Help me set up a pre-commit hook that runs accessibility tests before allowing commits. Use Husky and lint-staged."
```

**Expected Output:** Configuration for automated accessibility checking on commit.

---

### Stage 2: Active Development

**Goal:** Write accessible code from the start, not as an afterthought.

#### 2.1 Enable Accessible Development Mode

**When starting a new feature/component:**

```typescript
Skill({ skill: "accessibility:accessible-dev" })
```

**Then describe what you want to build:**

```
User: "Create a user registration form with:
- Email input (required)
- Password input with show/hide toggle (required)
- Confirm password input (required)
- Terms of service checkbox (required)
- Submit button
- Link to login page"
```

**Expected Behavior:**
- Claude generates semantic HTML (`<form>`, `<label>`, `<input>`, `<button>`)
- All inputs have associated labels via `for` attribute
- Required fields marked with `required` attribute AND visible indicator
- Password toggle button has `aria-label`
- Error messages use `role="alert"` and `aria-describedby`
- Form is fully keyboard operable
- Color contrast meets WCAG AA standards

**After completion, Claude will ask:**
```
"I've completed the registration form with accessible implementation. Would you like me to run a comprehensive accessibility review?"
```

**Respond:** `Yes` or `No` depending on whether you want immediate review.

---

#### 2.2 Building Specific Component Types

**For Forms:**
```typescript
Skill({ skill: "accessibility:accessible-dev" })

"Create a search form with autocomplete suggestions. The autocomplete dropdown should be keyboard navigable and screen reader friendly."
```

**For Modals/Dialogs:**
```typescript
Skill({ skill: "accessibility:accessible-dev" })

"Create a confirmation dialog modal with 'Cancel' and 'Confirm' buttons. The modal should trap focus and return focus to the trigger button when closed."
```

**For Navigation:**
```typescript
Skill({ skill: "accessibility:accessible-dev" })

"Create a primary navigation menu with dropdowns for 'Products', 'Resources', and 'Company'. Include skip links and support keyboard navigation."
```

**For Data Tables:**
```typescript
Skill({ skill: "accessibility:accessible-dev" })

"Create a data table showing user information (name, email, role, status) with sortable columns and row selection. Make it accessible for screen readers."
```

**For Tabs:**
```typescript
Skill({ skill: "accessibility:accessible-dev" })

"Create a tabbed interface for user settings with tabs for 'Profile', 'Security', and 'Notifications'. Support arrow key navigation."
```

**For Dropdowns/Select:**
```typescript
Skill({ skill: "accessibility:accessible-dev" })

"Create a custom dropdown select component with search filtering. Must be keyboard accessible and announce selections to screen readers."
```

---

#### 2.3 Quick Color Contrast Check During Development

**When adding new colors or themes:**

```typescript
Skill({ skill: "accessibility:contrast-checker" })

"Check color contrast in src/styles/theme.ts. Our primary button uses #5061ff on white background. Does this pass WCAG AA?"
```

**Expected Output:**
- Contrast ratio calculation (e.g., 4.67:1)
- PASS/FAIL for WCAG AA (4.5:1 for normal text, 3:1 for large text)
- Suggested color alternatives if failing
- Specific line numbers with violations

---

#### 2.4 Validate No Color-Only Indicators

**When implementing status indicators, errors, or required fields:**

```typescript
Skill({ skill: "accessibility:use-of-color" })

"Review src/components/StatusBadge.tsx and src/components/FormInput.tsx. Are there any cases where information is conveyed by color alone?"
```

**Expected Output:**
- List of violations (e.g., "Line 34: Error state shown only by red border")
- Recommended fixes (e.g., "Add error icon and text message")
- Code examples of proper implementation

---

#### 2.5 Check Link Text Clarity

**When adding navigation links, article cards, or CTAs:**

```typescript
Skill({ skill: "accessibility:link-purpose" })

"Check link text in src/components/BlogCard.tsx and src/pages/Articles.tsx. Are all links descriptive enough for screen reader users?"
```

**Expected Output:**
- List of generic link text ("Read more", "Click here", etc.)
- Context-aware recommendations
- Code examples using visually-hidden text or better link text

---

#### 2.6 Runtime Scanning with Playwright-A11y MCP

**NEW: Quick accessibility scans of live pages during development.**

The playwright-a11y MCP server provides instant accessibility feedback on running applications using Playwright + axe-core.

**When to use:**
- Quick spot-check during active development
- Validating fixes immediately after code changes
- Scanning pages without writing tests first
- Batch testing multiple pages at once

---

**Single URL Scan:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Quick scan of http://localhost:3000/dashboard for accessibility violations"
})
```

**What happens:**
1. Tester agent invokes `mcp__playwright_a11y__scan_url`
2. Playwright launches browser and navigates to URL
3. axe-core scans the live DOM
4. Violations returned instantly (no test file needed)
5. Report shows violations by severity with WCAG criteria

**Expected Output:**
```
Accessibility Scan Results:
URL: http://localhost:3000/dashboard

Critical Violations (2):
- color-contrast: Button text insufficient contrast (3.2:1, needs 4.5:1)
  Location: button.submit-btn
  WCAG: 1.4.3 Contrast (Minimum)

- label: Form input missing associated label
  Location: input#email
  WCAG: 3.3.2 Labels or Instructions

High Violations (1):
- link-name: Link has no accessible name
  Location: a.nav-link
  WCAG: 2.4.4 Link Purpose (In Context)

Summary: 3 violations found (2 Critical, 1 High)
```

---

**Batch Scan Multiple Pages:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Scan these URLs for accessibility: http://localhost:3000/home, http://localhost:3000/about, http://localhost:3000/contact"
})
```

**What happens:**
1. Tester agent invokes `mcp__playwright_a11y__scan_batch`
2. All URLs scanned in sequence
3. Aggregated report shows violations across all pages
4. Common patterns identified

**Expected Output:**
```
Batch Accessibility Scan Results:
Scanned 3 pages

Common Violations Across Pages:
- color-contrast: Found on 3/3 pages (Footer links)
- link-name: Found on 2/3 pages (Social media icons)
- heading-order: Found on 1/3 pages (About page)

Page-Specific Results:
/home: 2 violations (1 Critical, 1 High)
/about: 4 violations (1 Critical, 3 High)
/contact: 1 violation (1 High)

Recommendation: Fix footer link contrast across all pages first (affects all users).
```

---

**Scan Raw HTML (No Server Needed):**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Scan this HTML for accessibility: <div><button>Click</button></div>"
})
```

**What happens:**
- Scans HTML string without needing a live server
- Useful for testing snippets or generated HTML
- Returns same violation format

---

**Runtime Scanning vs Test Generation:**

| Runtime Scanning | Test Generation |
|------------------|-----------------|
| Instant feedback | Permanent regression tests |
| No test files created | Creates .spec.ts files |
| Good for quick checks | Good for CI/CD |
| Scans current state | Tests future changes |
| `scan_url` / `scan_batch` | `generate-a11y-tests` skill |

**Workflow combining both:**
1. **Develop** → Write code with accessible-dev
2. **Quick check** → Runtime scan with `scan_url`
3. **Fix** → Address violations found
4. **Re-scan** → Verify fixes with another `scan_url`
5. **Lock in** → Generate Playwright tests for regression prevention
6. **CI/CD** → Tests run automatically on every commit

---

**Saving Scan Reports:**

The tester agent can save scan results to files:

```typescript
Task({
  subagent_type: "tester",
  prompt: "Scan http://localhost:3000 and save the report to ./reports/pre-deployment-scan.json"
})
```

**What happens:**
- Scan runs as normal
- Results written to specified path
- Report includes all violation details, WCAG references, and remediation guidance

---

### Stage 3: Component/Feature Review

**Goal:** Comprehensive accessibility validation after completing a feature.

#### 3.1 Full Component Audit

**After completing a component:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Test src/components/RegistrationForm.tsx for WCAG 2.1 AA compliance. Check keyboard navigation, screen reader compatibility, form labels, error handling, and color contrast."
})
```

**Expected Output:**
1. Automated violations found (if any)
2. Manual testing recommendations
3. Keyboard navigation assessment
4. ARIA implementation review
5. **Prompt:** "Generate Playwright tests for this component?"

**Respond:** `Yes` to generate tests, `No` to skip

**If Yes:**
6. Test files created at `tests/accessibility/registration-form.spec.ts`
7. **Prompt:** "Run the tests now?"

**Respond:** `Yes` to run immediately, `No` to run manually later

---

#### 3.2 Page-Level Audit

**After completing a full page:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Audit the /checkout page for accessibility. Check heading hierarchy, landmark structure, keyboard navigation flow, and form accessibility."
})
```

**Expected Output:**
- Heading hierarchy validation (h1 → h2 → h3, no skipped levels)
- Landmark structure (`<header>`, `<nav>`, `<main>`, `<footer>`)
- Page title assessment
- Skip links verification
- Focus management for dynamic content
- **Offer to generate page-level Playwright tests**

---

#### 3.3 Feature-Level Audit

**After completing a multi-component feature:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Test the entire user authentication flow (login, registration, password reset) for accessibility. Ensure consistent patterns and WCAG compliance across all related components."
})
```

**Expected Output:**
- Cross-component consistency analysis
- User flow keyboard operability
- Error handling patterns
- Success/failure announcement strategies
- Focus management between pages/steps

---

### Stage 4: Testing & Validation

**Goal:** Automated and manual testing to catch regressions.

#### 4.1 Generate Comprehensive Test Suite

**Generate tests for multiple components at once:**

```typescript
Skill({ skill: "accessibility:generate-a11y-tests" })

"Generate Playwright accessibility tests for all components in src/components/forms/. Include axe-core scans, keyboard navigation tests, and ARIA validation."
```

**Expected Output:**
- Test files for each component
- Tests include:
  - axe-core automated scanning
  - Keyboard navigation tests
  - Focus management tests
  - ARIA attribute validation
  - Form-specific tests (label association, error handling)

---

#### 4.2 Generate Tests for Specific Scenarios

**Modal/Dialog Testing:**
```typescript
Skill({ skill: "accessibility:generate-a11y-tests" })

"Generate accessibility tests for src/components/ConfirmDialog.tsx. Focus on focus trapping, Escape key handling, and focus restoration on close."
```

**Navigation Testing:**
```typescript
Skill({ skill: "accessibility:generate-a11y-tests" })

"Generate accessibility tests for the main navigation. Test skip links, keyboard navigation through menus, and aria-current page indicators."
```

**Table Testing:**
```typescript
Skill({ skill: "accessibility:generate-a11y-tests" })

"Generate tests for src/components/DataTable.tsx. Verify table headers, sortable column accessibility, and row selection announcements."
```

---

#### 4.3 Run Tests

**Run all accessibility tests:**
```bash
npx playwright test tests/accessibility/
```

**Run specific test file:**
```bash
npx playwright test tests/accessibility/registration-form.spec.ts
```

**Run in UI mode (interactive debugging):**
```bash
npx playwright test tests/accessibility/ --ui
```

**Run in headed mode (see browser):**
```bash
npx playwright test tests/accessibility/ --headed
```

**Generate HTML report:**
```bash
npx playwright test tests/accessibility/ --reporter=html
npx playwright show-report
```

---

#### 4.4 Interpret Test Results

**When tests fail:**

```
Prompt: "The accessibility tests are failing with color-contrast violations. Show me the violations and how to fix them."
```

**Or:**

```typescript
Skill({ skill: "accessibility:contrast-checker" })

"The Playwright tests report color contrast failures on .btn-primary and .text-muted. Check these classes in src/styles/components.css and suggest fixes."
```

---

### Stage 5: Bug Fixing & Refactoring

**Goal:** Fix accessibility violations efficiently.

#### 5.1 Automatic Fix for Common Issues

```typescript
Skill({ skill: "accessibility:refactor" })

"Fix accessibility violations in src/components/LoginForm.tsx. The issues are:
1. Missing label associations
2. No error announcements
3. Color contrast failure on submit button"
```

**Expected Output:**
- Automatic code fixes applied
- Labels associated with inputs
- `aria-describedby` added for errors
- `role="alert"` added for error messages
- Color values adjusted to pass WCAG AA
- Summary of changes made

---

#### 5.2 Fix Specific Issue Types

**Fix Color Contrast:**
```typescript
Skill({ skill: "accessibility:contrast-checker" })

"Analyze src/styles/buttons.css. Fix any color contrast violations while preserving the overall design aesthetic. Suggest the closest compliant colors."
```

**Fix Color-Only Indicators:**
```typescript
Skill({ skill: "accessibility:use-of-color" })

"Fix color-only indicators in src/components/StatusLabel.tsx. Add icons or text to supplement color-coded statuses."
```

**Fix Link Text:**
```typescript
Skill({ skill: "accessibility:link-purpose" })

"Fix generic link text in src/pages/Blog.tsx. Make 'Read more' links descriptive for screen readers without changing visual design."
```

---

#### 5.3 Refactor Complex Component

**When a component needs significant accessibility improvements:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Review src/components/MegaMenu.tsx for accessibility issues, then use the refactor skill to fix all violations. The menu has keyboard navigation problems and missing ARIA."
})
```

**Expected Workflow:**
1. Tester agent identifies all issues
2. Automatically invokes refactor skill
3. Applies fixes
4. Re-tests to verify
5. Reports remaining manual fixes (if any)

---

### Stage 6: Pre-Commit & Code Review

**Goal:** Ensure accessibility before code reaches repository.

#### 6.1 Pre-Commit Quick Check

**Before committing:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Quick accessibility check on files I've modified: src/components/Button.tsx, src/components/Input.tsx, src/pages/Settings.tsx. Flag any critical violations."
})
```

**Expected Output:**
- Fast scan of changed files only
- Critical/High severity issues highlighted
- Recommendation to fix before commit or create follow-up ticket

---

#### 6.2 Generate Accessibility Report for PR

**When creating a pull request:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Generate an accessibility report for my PR changes. Compare accessibility compliance before and after my changes. Changed files: [list files]"
})
```

**Expected Output:**
- Markdown report suitable for PR description
- Accessibility improvements made
- New violations introduced (if any)
- Test coverage added
- Checklist of manual testing performed

---

#### 6.3 Run Tests Before Push

```bash
# Run accessibility tests
npm run test:a11y

# Or directly
npx playwright test tests/accessibility/

# If tests fail, get help
```

**If tests fail:**
```
Prompt: "Accessibility tests failed with [error message]. Help me understand and fix the issue."
```

---

### Stage 7: CI/CD & Deployment

**Goal:** Automated accessibility verification in CI/CD pipeline.

#### 7.1 Configure CI/CD Pipeline

**GitHub Actions:**

```
Prompt: "Create a GitHub Actions workflow that runs accessibility tests on every PR. The workflow should:
1. Run Playwright accessibility tests
2. Upload HTML report as artifact on failure
3. Comment on PR with summary of violations
4. Fail the build if Critical violations are found"
```

**GitLab CI:**
```
Prompt: "Create a GitLab CI configuration that runs accessibility tests in the test stage. Upload reports as artifacts and fail pipeline on Critical violations."
```

---

#### 7.2 Pre-Deployment Audit

**Before deploying to production:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Perform a final accessibility audit before production deployment. Test all critical user paths: homepage, login, registration, checkout. Generate a compliance report."
})
```

**Expected Output:**
- Complete accessibility compliance report
- Critical path verification
- WCAG 2.1 AA conformance statement
- Known issues and workarounds
- Recommendation to deploy or hold

---

### Stage 8: Maintenance & Audits

**Goal:** Ongoing accessibility monitoring and improvement.

#### 8.1 Quarterly Accessibility Audit

**Every 3 months:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Comprehensive accessibility audit of the entire application. Compare against previous audit from [date]. Identify any regressions or new violations. Generate executive summary."
})
```

**Expected Output:**
- Full audit report
- Comparison with previous audit
- New violations introduced
- Resolved violations
- Accessibility score trend
- Recommended priorities for next quarter

---

#### 8.2 New Dependency Audit

**When adding third-party components:**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Audit the new React date picker library we're considering (react-datepicker). Check its accessibility: keyboard navigation, screen reader announcements, ARIA implementation. Recommend if we should use it."
})
```

**Expected Output:**
- Accessibility assessment of library
- Known issues or limitations
- Recommendation (use as-is, use with modifications, or find alternative)
- Implementation guidance for accessible usage

---

#### 8.3 Accessibility Regression Testing

**After major refactoring or updates:**

```bash
# Run full test suite
npx playwright test tests/accessibility/

# Review results
npx playwright show-report
```

**If regressions found:**
```typescript
Skill({ skill: "accessibility:refactor" })

"Fix accessibility regressions introduced in [commit/PR]. The following components now have violations: [list]"
```

---

## Common Scenarios

### Scenario 1: "I'm starting a new React component"

```typescript
// Step 1: Enable accessible development
Skill({ skill: "accessibility:accessible-dev" })

// Step 2: Describe component
"Create a user profile card component showing avatar, name, bio, and 'Edit Profile' button. Make it keyboard navigable."

// Step 3: Claude generates accessible code

// Step 4: Claude asks for review
// Respond: "Yes"

// Step 5: Tester runs audit

// Step 6: Generate tests when prompted
// Respond: "Yes"

// Step 7: Tests created

// Step 8: Run tests
// Respond: "Yes"

// Done! Component is accessible and tested.
```

---

### Scenario 2: "I have an existing component with accessibility issues"

```typescript
// Step 1: Audit the component
Task({
  subagent_type: "tester",
  prompt: "Test src/components/ProductCard.tsx for accessibility issues. It's missing labels and has color contrast problems."
})

// Step 2: Review violations reported

// Step 3: Fix automatically
Skill({ skill: "accessibility:refactor" })
"Fix all accessibility violations in src/components/ProductCard.tsx"

// Step 4: Generate tests
Skill({ skill: "accessibility:generate-a11y-tests" })
"Generate tests for src/components/ProductCard.tsx"

// Step 5: Run tests
// Run: npx playwright test tests/accessibility/product-card.spec.ts

// Step 6: Verify all passing
```

---

### Scenario 3: "Our color palette is not WCAG compliant"

```typescript
// Step 1: Analyze current palette
Skill({ skill: "accessibility:contrast-checker" })
"Analyze all colors in src/styles/colors.css and src/styles/theme.ts. Check every foreground/background combination we use."

// Step 2: Review violations

// Step 3: Get compliant alternatives
"For each failing combination, suggest the closest WCAG AA compliant color that maintains our brand aesthetic."

// Step 4: Apply fixes
Skill({ skill: "accessibility:refactor" })
"Update src/styles/colors.css with the WCAG-compliant color palette suggested above."

// Step 5: Verify with tests
npx playwright test tests/accessibility/ --grep "contrast"
```

---

### Scenario 4: "I need to audit an entire page before launch"

```typescript
// Step 1: Comprehensive page audit
Task({
  subagent_type: "tester",
  prompt: "Perform complete accessibility audit of /product-details page including:
  - Heading hierarchy
  - Landmark structure
  - Keyboard navigation
  - Image alt text
  - Color contrast
  - Form accessibility (add to cart, quantity selector)
  - Mobile accessibility
  Generate launch readiness report."
})

// Step 2: Review audit results

// Step 3: Generate test suite
Skill({ skill: "accessibility:generate-a11y-tests" })
"Generate comprehensive test suite for /product-details page covering all components and interactions."

// Step 4: Run tests
npx playwright test tests/accessibility/product-details-page.spec.ts

// Step 5: Fix any failures

// Step 6: Re-run tests

// Step 7: Launch!
```

---

### Scenario 5: "Tests are failing in CI but passing locally"

```
Prompt: "Accessibility tests pass locally but fail in CI with 'color-contrast' violations. The error mentions buttons in the header. Help me debug this."

// Claude will:
// 1. Check viewport differences (CI vs local)
// 2. Check color rendering differences
// 3. Suggest adding explicit viewport to tests
// 4. Recommend running tests with --headed locally to reproduce
```

---

### Scenario 6: "Need accessibility checklist for code review"

```
Prompt: "Generate an accessibility code review checklist for React components. I want to use this when reviewing PRs."
```

**Expected Output:**
```markdown
## Accessibility Code Review Checklist

### Semantic HTML
- [ ] Uses semantic elements (<button>, <nav>, <main>, etc.)
- [ ] No divs/spans for interactive elements
- [ ] Proper heading hierarchy (h1 → h2 → h3)

### Keyboard Accessibility
- [ ] All interactive elements keyboard accessible
- [ ] Logical tab order
- [ ] Visible focus indicators
- [ ] No keyboard traps (except intentional in modals)

### Screen Reader Support
- [ ] All images have alt text
- [ ] Forms have associated labels
- [ ] Error messages use aria-live or role="alert"
- [ ] Dynamic content changes announced

### ARIA
- [ ] ARIA used only when semantic HTML insufficient
- [ ] Correct ARIA roles
- [ ] ARIA states managed (aria-expanded, aria-selected)
- [ ] No ARIA on semantic elements (e.g., no role="button" on <button>)

### Color & Contrast
- [ ] Color contrast WCAG AA (4.5:1 normal, 3:1 large text)
- [ ] Information not conveyed by color alone
- [ ] Focus indicators have sufficient contrast

### Forms
- [ ] All inputs have labels
- [ ] Required fields marked (required attribute + visual)
- [ ] Error messages associated with fields (aria-describedby)
- [ ] Validation errors announced

### Tests
- [ ] Playwright accessibility tests added
- [ ] Tests passing locally
- [ ] axe-core scan included
```

---

### Scenario 7: "Quick runtime scan during development"

**NEW: Using playwright-a11y MCP for instant feedback**

```typescript
// Scenario: You're developing a feature and want instant accessibility feedback

// Step 1: Start your dev server
// Run: npm run dev (or equivalent)
// Server running at http://localhost:3000

// Step 2: Quick scan of current page
Task({
  subagent_type: "tester",
  prompt: "Quick scan http://localhost:3000/dashboard for accessibility violations"
})

// Step 3: Review instant results
// - No test files needed
// - axe-core runs in real browser
// - Violations shown immediately

// Step 4: Fix critical issues
"Fix the color contrast violation on the submit button"

// Step 5: Re-scan to verify
Task({
  subagent_type: "tester",
  prompt: "Re-scan http://localhost:3000/dashboard to verify fixes"
})

// Step 6: Once all violations fixed, generate permanent tests
Skill({ skill: "accessibility:generate-a11y-tests" })
"Generate tests for the dashboard page"

// Done! Fast feedback loop for rapid iteration.
```

**Benefits of runtime scanning:**
- ⚡ Instant feedback (no test writing needed first)
- 🌐 Tests real browser rendering
- 🔄 Perfect for iterative development
- 📊 See exactly what users will experience

**When to use:**
- During active development
- Validating quick fixes
- Before committing changes
- Testing pages that are hard to test statically

---

### Scenario 8: "Batch scan before deployment"

**Scan multiple pages at once for pre-deployment verification**

```typescript
// Step 1: Batch scan critical pages
Task({
  subagent_type: "tester",
  prompt: "Scan these pages for accessibility before deployment: http://localhost:3000/, http://localhost:3000/products, http://localhost:3000/checkout, http://localhost:3000/account"
})

// Step 2: Review aggregated results
// Common violations across pages identified
// Page-specific issues highlighted
// Priority recommendations provided

// Step 3: Fix common violations first
Skill({ skill: "accessibility:refactor" })
"Fix the footer link contrast issue that appears on all pages"

// Step 4: Re-scan to verify
Task({
  subagent_type: "tester",
  prompt: "Re-scan the same 4 pages to verify footer fixes"
})

// Step 5: Generate deployment report
Task({
  subagent_type: "tester",
  prompt: "Generate a pre-deployment accessibility report and save to ./reports/deployment-scan-2026-02-07.json"
})

// Step 6: Review report shows all pages passing ✅
// Ready to deploy!
```

**Batch scanning advantages:**
- 🎯 Tests multiple pages in one command
- 📈 Identifies patterns across site
- ⏱️ Faster than individual scans
- 📋 Generates comprehensive reports

---

## Skill & Agent Reference

### Skills

#### `accessibility:accessible-dev`
**Purpose:** Proactive accessible code generation
**When to Use:** Starting new components/features
**Tools:** Read, Glob, Grep, Edit, Skill
**Expected Outcome:** Accessible code from the start

**Usage Pattern:**
```typescript
Skill({ skill: "accessibility:accessible-dev" })
// Then describe what to build
```

---

#### `accessibility:contrast-checker`
**Purpose:** Color contrast analysis
**When to Use:** Checking/fixing color combinations
**MCP Tools:** calculate_contrast_ratio, analyze_color_pair, suggest_accessible_color
**Expected Outcome:** Contrast ratios, PASS/FAIL, suggested fixes

**Usage Pattern:**
```typescript
Skill({ skill: "accessibility:contrast-checker" })
// Specify files or colors to check
```

---

#### `accessibility:use-of-color`
**Purpose:** Detect color-only indicators
**When to Use:** Reviewing status, errors, required fields
**Expected Outcome:** List of violations with fix suggestions

**Usage Pattern:**
```typescript
Skill({ skill: "accessibility:use-of-color" })
// Specify files to check
```

---

#### `accessibility:link-purpose`
**Purpose:** Validate link text clarity
**When to Use:** Reviewing navigation, cards, CTAs
**Expected Outcome:** Generic links identified with better alternatives

**Usage Pattern:**
```typescript
Skill({ skill: "accessibility:link-purpose" })
// Specify files to check
```

---

#### `accessibility:refactor`
**Purpose:** Automatically fix accessibility violations
**When to Use:** After identifying issues, bulk fixing
**Tools:** Read, Grep, Edit, Skill
**Expected Outcome:** Code updated with fixes applied

**Usage Pattern:**
```typescript
Skill({ skill: "accessibility:refactor" })
// Describe issues to fix or specify files
```

---

#### `accessibility:generate-a11y-tests`
**Purpose:** Generate Playwright accessibility tests
**When to Use:** After building components, creating test coverage
**Tools:** Read, Glob, Grep, Write
**Expected Outcome:** Test files with axe-core + manual tests

**Usage Pattern:**
```typescript
Skill({ skill: "accessibility:generate-a11y-tests" })
// Specify components/pages to test
```

---

### Agents

#### `tester` (accessibility-tester)
**Purpose:** Comprehensive accessibility auditing
**When to Use:** Reviewing components, pages, or full application
**Tools:** Read, Grep, Glob, Bash, Skill, MCP tools
**Model:** haiku (efficient)
**Workflow:**
1. Analyzes code
2. Invokes specialized skills
3. Generates report
4. Offers to generate tests
5. Offers to run tests
6. Reports results

**Usage Pattern:**
```typescript
Task({
  subagent_type: "tester",
  prompt: "Test [path/component/page] for accessibility"
})
```

**Alternative invocation (short form):**
```typescript
Task({
  subagent_type: "tester",
  prompt: "Audit src/components/Button.tsx"
})
```

---

## Troubleshooting

### Issue: Plugin not found

```bash
# Check installed plugins
claude plugin list

# Reinstall
claude plugin install accessibility@deepakkamboj

# Sync plugins
claude plugin sync
```

---

### Issue: MCP tools not available

```bash
# Check MCP server status
claude mcp status

# Restart MCP server
claude mcp restart accessibility

# View logs
claude mcp logs accessibility
```

---

### Issue: Skills not working

```
Prompt: "List all available accessibility skills"

# Should show:
# - accessibility:accessible-dev
# - accessibility:contrast-checker
# - accessibility:use-of-color
# - accessibility:link-purpose
# - accessibility:refactor
# - accessibility:generate-a11y-tests
```

If skills missing:
```bash
# Reload plugin
claude plugin reload accessibility
```

---

### Issue: Tests failing with "Cannot find module @playwright/test"

```bash
# Install Playwright
npm install -D @playwright/test @axe-core/playwright

# Install browsers
npx playwright install
```

---

### Issue: Tests passing locally but failing in CI

**Common causes:**
1. **Viewport differences** - CI uses different screen size
2. **Color rendering** - Slight color variations
3. **Timing issues** - CI slower, needs longer waits

**Solutions:**
```typescript
// Add explicit viewport to playwright.config.ts
use: {
  viewport: { width: 1280, height: 720 },
}

// Add longer timeouts for CI
timeout: process.env.CI ? 60000 : 30000,
```

---

### Issue: Too many violations overwhelming me

**Strategy: Prioritize by severity**

```typescript
Task({
  subagent_type: "tester",
  prompt: "Audit src/ for accessibility violations. Only report Critical and High severity issues. Group by pattern, don't report every occurrence."
})
```

**Then fix in order:**
1. Critical (blocks users completely)
2. High (major barriers)
3. Medium (minor barriers)
4. Low (best practice improvements)

---

### Issue: Third-party component not accessible

**Options:**

1. **Find alternative library**
```
Prompt: "The 'react-select' library has accessibility issues. Suggest alternative accessible React select components."
```

2. **Wrap and enhance**
```
Prompt: "Create an accessible wrapper around the XYZ component that adds proper ARIA and keyboard support."
```

3. **Report upstream and use workaround**
```
Prompt: "Document the accessibility issues in XYZ library and create a workaround using ARIA live regions."
```

---

### Issue: Don't know where to start with accessibility

**Start here:**

```typescript
// Step 1: Learn the standards
"Read ACCESSIBILITY_STANDARDS.md and explain the top 5 most important WCAG rules for web forms."

// Step 2: Audit one component
Task({
  subagent_type: "tester",
  prompt: "Teach me accessibility by auditing src/components/Button.tsx. Explain each violation and why it matters."
})

// Step 3: Fix with explanations
Skill({ skill: "accessibility:refactor" })
"Fix src/components/Button.tsx and explain each change you make."

// Step 4: Generate tests
Skill({ skill: "accessibility:generate-a11y-tests" })
"Generate tests for Button.tsx and explain what each test verifies."

// Step 5: Apply to next component
// Repeat process with increasing independence
```

---

## Additional Resources

- **Standards Reference:** [ACCESSIBILITY_STANDARDS.md](ACCESSIBILITY_STANDARDS.md)
- **Playwright Integration:** [plugins/accessibility/docs/PLAYWRIGHT_INTEGRATION.md](plugins/accessibility/docs/PLAYWRIGHT_INTEGRATION.md)
- **WCAG Quick Reference:** https://www.w3.org/WAI/WCAG21/quickref/
- **ARIA Patterns:** https://www.w3.org/WAI/ARIA/apg/

---

## Summary: Development Stage → Command Quick Reference

| Stage | Primary Command |
|-------|----------------|
| **Setup** | Read `ACCESSIBILITY_STANDARDS.md`, Audit codebase with `tester` agent |
| **Development** | `Skill({ skill: "accessibility:accessible-dev" })` + describe component |
| **Quick Checks** | `contrast-checker`, `use-of-color`, `link-purpose` skills |
| **Review** | `Task({ subagent_type: "tester", prompt: "Test [path]" })` |
| **Testing** | `generate-a11y-tests` skill → `npx playwright test` |
| **Fixing** | `accessibility:refactor` skill or manual fixes |
| **Pre-Commit** | `npx playwright test tests/accessibility/` |
| **CI/CD** | Automated Playwright tests in pipeline |
| **Maintenance** | Quarterly audits with `tester` agent |

---

**Pro Tip:** Start every new feature with `accessible-dev` skill active. Accessibility is easier to build in than to retrofit!
