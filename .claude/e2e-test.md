# E2E Tests

Playwright-based end-to-end tests that run against a deployed environment (alpha, PR preview, or local). Tests interact with real browser sessions authenticated via Passport.

## Project Naming

| Pattern | Example |
|---------|---------|
| `WebSpa.E2ETests` | `Banking/test/WebSpa.E2ETests` |
| `Passport.E2ETests` | `Passport/test/Passport.E2ETests` |

## Tech Stack

- **Framework:** Playwright (`@playwright/test`)
- **Language:** TypeScript
- **Accessibility:** `axe-playwright` + `axe-core`
- **APM:** `dd-trace` (DataDog trace injection)
- **Shared helpers:** `@nbs/ng-testing` (from `Framework.Web.AngularTesting`)
- **Build artifact:** `.esproj` / `.njsproj` (VS Node.js project type)

## Project Structure

```
WebSpa.E2ETests/
  tests/                  # Test specs grouped by feature
    merchant-accounts/
    financial-accounts/
    payment-methods/
    apple-pay/
    calendars/
    transaction-activity/
    no-access-tests.spec.ts
  page-objects/           # Page Object Model classes
  test-const/             # Shared constants (URLs, selectors, test data)
  *-fixtures.ts           # Playwright fixtures (auth setup, test context)
  playwright.config.ts    # Playwright configuration
  tsconfig.json
```

## Environments

Configured via environment variables in `playwright.config.ts`:

| Env Var | Default | Purpose |
|---------|---------|---------|
| `E2E_BASE_URL` | local/alpha URL | SPA base URL under test |
| `PASSPORT_WEB_URL` | login-alpha1 | Auth provider URL |
| `USE_ALPHA_BASEURL` | — | Switch to alpha environment |

## Run Commands

```bash
cd test/WebSpa.E2ETests
npm install
npx playwright install    # Install browsers (first time)

npm test                  # Full test suite
npm run test-verbose      # Full suite with debug output (DEBUG=pw:api)
npm run test-alpha        # Run against alpha environment
npm run test-inspector    # Interactive Playwright Inspector (PWDEBUG=1)

# PR-scoped subsets (run specific feature folder)
npm run test-pr-merchant-accounts
npm run test-pr-financial-accounts
npm run test-pr-payment-methods
npm run test-pr-apple-pay
npm run test-pr-calendars
npm run test-pr-transaction-activity
npm run test-pr-no-access

# Local PR env (custom URL)
npm run test-local-pr
```

## CI Integration

E2E tests run in `azure-pipelines-pr.yml` via NPS UAT pipeline:

```
azure-pipelines-nps-uat-tests.yml   # Banking NPS UAT
```

JUnit reports merged via:
```bash
npm run merge-junit-reports   # → junit/junit.xml
```

## Prerequisites

E2E tests require full service stack running. Use Xlr8 Tools to start all deps:

```powershell
Invoke-Xlr8GoToDirectory <ServiceName>
Invoke-Xlr8ServiceRunner rm *
Invoke-Xlr8ServiceRunner add . deps   # includes this service + all deps
```

See [dependencies.md](dependencies.md) for each service's dep list.

## nbs-ag-grid E2E Patterns (CSP-only)

> **Scope:** Only CSP tests AG Grid sorting, inline filters, and field-search. All other repos (Banking, Checkout, Payments, Remittance, Tenant, Passport) only test basic grid presence — rows visible, loading spinner, accessibility. Patterns below are CSP-originated; apply to other repos if they add similar coverage.

### Sorting

AG Grid header cells expose `aria-sort` attribute. Two styles exist in CSP:

**Style A — role-based locators** (`search-tenants.spec.ts`):
```typescript
const header = datatable.getByRole('columnheader', { name: 'Name' });
const sortButton = header.getByRole('button', { name: /sort/i }).first();
await sortButton.click();
await expect(header).toHaveAttribute('aria-sort', 'ascending');
```

**Style B — CSS selector + page object helpers** (`search-tenants-sorting.spec.ts`):
```typescript
// PO maps display names → col-id via colIdMap
await tenantPage.clickSortHeader('Name');   // clicks .ag-header-cell[col-id="name"]
const dir = await tenantPage.getSortDirection('Name');  // reads aria-sort attribute
expect(dir).toBeTruthy();
```

Style B also monitors for NG0600 console errors during sort.

### Inline Filters (`[inlineFilters]="true"`)

Inline filter row renders below header text. Each column has a clickable filter trigger.

**Locators:**
- Filter trigger: `.ag-header-cell[col-id="products"] .inline-filter-clickable`
- Filter popup: `page.locator('.ag-popup')` (rendered outside grid DOM)
- Select filter options: `filterPopup.locator('mat-list-option')`
- Active filter indicator: header cell gets CSS class `filtered`
- Clear button: `.inline-filter-clear`

**Critical:** Popup stays open after selecting options. Must press `Escape` (or click outside) to dismiss — header display and CSS class only update after popup closes.

```typescript
// Open filter
await productsHeader.locator('.inline-filter-clickable').click();
const popup = page.locator('.ag-popup');
await expect(popup).toBeVisible();

// Select option
await popup.locator('mat-list-option').first().click();

// Close popup — required for filter to apply
await page.keyboard.press('Escape');
await expect(popup).not.toBeVisible();

// Assert
await expect(productsHeader).toHaveClass(/filtered/);
```

### nbs-field-search Pitfalls

`fieldSearchButton` (`getByRole('button')` inside `nbs-field-search`) resolves to multiple elements when chips exist — chip remove icons also have `role="button"`. Causes Playwright strict mode violations.

**Workaround:** Use `enterKeyword(value)` (types + presses Enter) instead of `performSearch()` (clicks button) when chips may be present. Wait for rows with `await expect(rows.first()).toBeVisible({ timeout: 15000 })` instead of asserting spinner hidden (spinner may never appear if search is fast).

## Per-Repo Notes

| Repo | E2E Project | Notes |
|------|-------------|-------|
| Banking | `WebSpa.E2ETests` | Most complete — 6 feature suites; grid tests = presence only |
| Checkout | `WebSpa.E2ETests` | Grid tests = presence only |
| Payments | `WebSpa.E2ETests` | Grid tests = loading spinner only |
| Remittance | `WebSpa.E2ETests` | Grid tests = row presence + accessibility |
| Encryption | `WebSpa.E2ETests` | — |
| EventHub | `WebSpa.E2ETests` | — |
| Tenant | `WebSpa.E2ETests` | Change history row presence only |
| CustomerServicePortal | `WebSpa.E2ETests` | **Only repo with AG Grid sort, inline filter, field-search tests**; also people search, person dashboard |
| Passport | `Passport.E2ETests` | Covers `AdminWeb`, `Web`, `Common`; grid = row presence + checkbox selection |
