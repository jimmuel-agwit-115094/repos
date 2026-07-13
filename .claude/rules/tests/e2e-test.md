# E2E Tests

Playwright-based end-to-end tests against a deployed environment (alpha, PR preview, or local). Auth via Passport. All repos share the same `@nbs/ng-testing` base framework.

## Project Naming & Location

| Pattern | Example |
|---------|---------|
| `WebSpa.E2ETests` | `Banking/test/WebSpa.E2ETests` |
| `Passport.E2ETests` | `Passport/test/Passport.E2ETests` |

## Tech Stack

- **Framework:** `@playwright/test` v1.61.1
- **Language:** TypeScript (strict mode)
- **Shared base:** `@nbs/ng-testing/playwright-v2` — base classes, fixtures, login helpers
- **Accessibility:** `axe-playwright@1.2.3` + `axe-core@4.9.1`
- **APM:** `dd-trace` (DataDog trace injection)
- **Build artifact:** `.esproj` / `.njsproj` (VS Node.js project type)

## Project Structure

```
WebSpa.E2ETests/
  tests/                    # Specs grouped by feature
    people-search/
    person-dashboard/
    search-users/
    search-tenants/
    recent-searches/
  page-objects/             # Page Object Model classes ({component}.po.ts)
  test-const/               # Shared constants, fixtures, seeded data IDs
  playwright.config.ts
  package.json
  tsconfig.json
  junit/                    # JUnit report output
```

## Environments & Config

**`playwright.config.ts` base URL precedence:**
1. `process.env.E2E_BASE_URL` (CLI override)
2. `process.env.USE_ALPHA_BASEURL` → alpha2 URL
3. Default → `https://localhost:{port}/web/{service}/`

**Key config values (CSP example):**

```typescript
import { nbsPlaywrightTestConfig } from '@nbs/ng-testing/playwright';

const config = nbsPlaywrightTestConfig;
config.use!.baseURL = determineBaseUrl();
config.use!.testIdAttribute = 'data-test-id';  // getByTestId() maps to this attr
config.timeout = 35_000;
config.retries = 2;
config.forbidOnly = !!(process.env.CI || process.env.TF_BUILD);
```

## Run Commands

```bash
cd test/WebSpa.E2ETests
npm install
npx playwright install    # First time only

npm test                        # Full suite
npm run test-verbose            # DEBUG=pw:api
npm run test-alpha              # USE_ALPHA_BASEURL=1
npm run test-inspector          # PWDEBUG=1, --workers=1
npm run test-pr-{feature}       # Feature-scoped PR subset
npm run merge-junit-reports     # → junit/junit.xml
```

## Prerequisites — Starting Services

```powershell
Invoke-Xlr8GoToDirectory <ServiceName>
Invoke-Xlr8ServiceRunner rm *
Invoke-Xlr8ServiceRunner add . deps   # this service + all deps
```

See [dependencies.md](../microservice-setup/dependencies.md) for each service's dep list.

---

## Page Object Model

### Class Hierarchy

```
@nbs/ng-testing/playwright-v2
└── BasePage                     # All CSP POs extend this
    ├── component: Locator        # Scoped to Angular component selector
    ├── page: Page
    ├── snackbarMessage           # NBS snackbar
    └── shouldBeAccessible(n)     # Runs axe; allows n violations
```

### PO File Pattern

```typescript
// page-objects/search-users/search-users-po.ts
import { Page, BasePage } from '@nbs/ng-testing/playwright-v2';

export class SearchUsersPage extends BasePage {
  // All locators: readonly, declared on class (not in methods)
  readonly header        = this.component.getByRole('heading', { name: 'Search', exact: true });
  readonly userSearch    = this.component.getByTestId('input_searchFilter');
  readonly searchButton  = this.component.getByTestId('nbsButton_search');
  readonly progressBar   = this.component.getByTestId('progress-bar');
  readonly datatable     = this.component.getByTestId('user-table');

  // CDK overlays / dialogs: scope to this.page, not this.component
  readonly dialog        = this.page.locator('mat-dialog-container');

  constructor(page: Page) {
    super(page, 'app-search-tab');  // Angular component selector for scoping
  }

  async navigate() {
    await this.page.goto('search');
    await this.waitFor();
  }

  async clickSearch() {
    await this.searchButton.click();
    await this.progressBar.waitFor({ state: 'detached' });
    await this.datatable.waitFor();
  }
}
```

### Locator Strategy (priority order)

| Priority | Strategy | When |
|----------|----------|------|
| 1 | `getByTestId('data-test-id-value')` | Primary — always prefer |
| 2 | `getByRole('button', { name: ... })` | Semantic elements without test IDs |
| 3 | CSS `.ag-header-cell[col-id="name"]` | AG Grid internals only |
| 4 | `:has-text()` | Last resort; combine with `.first()` |

**Test ID naming conventions (`data-test-id`):**

| Element | Pattern | Example |
|---------|---------|---------|
| Button | `npds_button_{name}` or `nbsButton_{name}` | `npds_button_search` |
| Input | `input_{name}` | `input_searchFilter` |
| Icon | `matIcon_{name}` | `matIcon_warningBlankSearch` |
| Form field | `matFormField_{name}` | `matFormField_search` |
| Table/Grid | `{name}-table` or `data-table` | `user-table`, `data-table` |
| Container | `div_{name}` | `div_noResultsState` |
| Progress | `progress-bar` | `progress-bar` |

---

## Authentication

Import from `@nbs/ng-testing/playwright-v2`:

```typescript
import { test, expect, loginAsFullAccessNbsUser } from '@nbs/ng-testing/playwright-v2';

// Available helpers
loginAsFullAccessNbsUser(page)   // Default CSR — use for most tests
loginAsReadOnlyNbsUser(page)     // Read-only CSR role
loginAsNoAccessNbsUser(page)     // 403 guard testing
login(page, userId?: string)     // Custom user by ID
```

Called first in every `test.beforeEach`. Uses Passport test-login API — no password entry.

---

## Test Spec Pattern

```typescript
import { test, expect, loginAsFullAccessNbsUser } from '@nbs/ng-testing/playwright-v2';
import { SearchUsersPage } from 'page-objects/search-users/search-users-po';

test.use({ subdomain: 'andromedauni' });  // Tenant scope when needed

let searchUserPage: SearchUsersPage;
let keyword: string;  // Outer scope for afterEach cleanup

test.describe('User Search', () => {
  test.beforeEach(async ({ page }) => {
    await loginAsFullAccessNbsUser(page);  // ALWAYS first
    searchUserPage = new SearchUsersPage(page);
    await searchUserPage.navigate();
  });

  test.afterEach(async () => {
    if (keyword) {
      await searchUserPage.deleteRecentSearchKeywordUsingGetRecentSearchRow(keyword);
    }
  });

  test('should show results for valid keyword', async () => {
    keyword = 'test-user';
    await searchUserPage.searchUser(keyword);
    await expect(searchUserPage.datatable).toBeVisible();
  });
});
```

---

## Waiting & Timing

**Standard flow:**

```typescript
// After triggering an action:
await this.progressBar.waitFor({ state: 'detached', timeout: 10000 });
await this.datatable.waitFor();  // auto-retries until visible

// Wait for one of two states (results OR empty):
await this.rows.first().or(this.noRowsOverlay).waitFor();

// Wait for URL change:
await page.waitForURL(new RegExp(`person-dashboard/${personId}`), { timeout: 30000 });

// Wait for DOM condition (custom state):
await page.waitForFunction(() => {
  return document.querySelector('[data-test-id="div_noResultsState"]')
      || document.querySelector('[data-test-id="div_searchResults"]');
}, { timeout: 30000 });
```

**Never use `waitForTimeout()` except for Angular change-detection gaps** (brief `500ms` max).

---

## Accessibility Testing

```typescript
// Inherited from BasePage — call after page is fully loaded
await searchUserPage.shouldBeAccessible();     // Zero violations
await searchUserPage.shouldBeAccessible(1);    // Allow 1 known violation
```

---

## Test Constants / Fixture Data

Seeded fixtures live in specific tenants. Document in `test-const/`:

```typescript
// test-const/const-person-dashboard.ts
export const TENANT_SUBDOMAIN = 'andromedauni';  // WHERE fixtures are seeded

export const PERSON_FIXTURES = {
  P_STU_EXT: {
    id: 'P-STU-EXT',
    personId: '6a0d7cdf873c95a48f1aefcf',
    fullName: 'P-STU-EXT E2ETest',
    externalId: 'S-100-024',
    notes: 'Student, full data'
  },
  // ...
};

export const CSR_FIXTURES = {
  CSR_INST:        { username: 'CSR-INSTITUTION', notes: 'External Institution CSR' },
  CSR_INST_NOPERM: { username: 'CSR-INST-NOPERM', notes: 'No people_search permission' },
};
```

---

## nbs-ag-grid E2E Patterns (CSP-only)

> **Scope:** Only CSP tests AG Grid sorting, inline filters, and field-search. All other repos (Banking, Checkout, Payments, Remittance, Tenant, Passport) only test basic grid presence — rows visible, loading spinner, accessibility.

### Sorting

AG Grid header cells expose `aria-sort`. Two styles exist in CSP:

**Style A — role-based** (`search-tenants.spec.ts`):
```typescript
const header = datatable.getByRole('columnheader', { name: 'Name' });
const sortButton = header.getByRole('button', { name: /sort/i }).first();
await sortButton.click();
await expect(header).toHaveAttribute('aria-sort', 'ascending');
```

**Style B — CSS selector + PO helpers** (`search-tenants-sorting.spec.ts`):
```typescript
// PO maps display names → col-id via colIdMap
await tenantPage.clickSortHeader('Name');          // clicks .ag-header-cell[col-id="name"]
const dir = await tenantPage.getSortDirection('Name');  // reads aria-sort
expect(dir).toBeTruthy();
```

Monitor for NG0600 during sort:
```typescript
const consoleErrors: string[] = [];
page.on('console', msg => {
  if (msg.type() === 'error' && msg.text().includes('NG0600')) consoleErrors.push(msg.text());
});
// ... after test:
expect(consoleErrors).toHaveLength(0);
```

### Inline Filters (`[inlineFilters]="true"`)

```typescript
// Filter trigger per column
await productsHeader.locator('.inline-filter-clickable').click();
const popup = page.locator('.ag-popup');   // outside grid DOM — use page not component
await expect(popup).toBeVisible();

await popup.locator('mat-list-option').first().click();

// REQUIRED: close popup before asserting filter applied
await page.keyboard.press('Escape');
await expect(popup).not.toBeVisible();

await expect(productsHeader).toHaveClass(/filtered/);
await productsHeader.locator('.inline-filter-clear').click();  // reset
```

### nbs-field-search Pitfalls

`getByRole('button')` inside `nbs-field-search` resolves to **multiple elements** when chips exist (chip remove icons also have `role="button"`) → Playwright strict-mode violation.

**Workaround:** Use `enterKeyword(value)` (types + Enter) instead of `performSearch()` (clicks button) when chips may be present. Wait for rows with `await expect(rows.first()).toBeVisible({ timeout: 15000 })` rather than asserting spinner hidden.

---

## Common Pitfalls & Gotchas

| Pitfall | Fix |
|---------|-----|
| Wrong tenant subdomain → no fixture data → CI timeout | Always use `TENANT_SUBDOMAIN` constant; verify fixtures seeded in DbScripts |
| Strict mode: multiple matches on `:has-text()` | Append `.first()` or iterate with `.nth(i)` |
| Stale locators across page reload | Always query fresh after navigation — never cache element handles |
| Missing `afterEach` cleanup for recent searches | Store keyword in outer `let`; always delete in afterEach |
| AG Grid card order not guaranteed across reloads | Sort deterministically in PO `getOrderedCards()` method |
| Multi-tenant row sort keys use minimum tenant name | Extract `allInnerTexts()`, sort locally, take `[0]` |
| NG0600 console error on grid sort | Hook `page.on('console')` to detect and fail early |
| `.only` in committed test | Blocked by `forbidOnly` in CI — fails the pipeline |

---

## CI Integration

```yaml
# azure-pipelines-pr.yml — E2E runs in NPS UAT pipeline
azure-pipelines-nps-uat-tests.yml   # Banking example
```

JUnit reports merged:
```bash
npm run merge-junit-reports   # → junit/junit.xml
```

---

## Per-Repo Notes

| Repo | E2E Project | Grid test depth |
|------|-------------|-----------------|
| Banking | `WebSpa.E2ETests` | Row presence only |
| Checkout | `WebSpa.E2ETests` | Row presence only |
| Payments | `WebSpa.E2ETests` | Loading spinner only |
| Remittance | `WebSpa.E2ETests` | Row presence + accessibility |
| Encryption | `WebSpa.E2ETests` | — |
| EventHub | `WebSpa.E2ETests` | — |
| xlr8-app-tenant | `WebSpa.E2ETests` | Change history row presence only |
| CustomerServicePortal | `WebSpa.E2ETests` | **Full** — AG Grid sort, inline filter, field-search, people search, person dashboard |
| Passport | `Passport.E2ETests` | Row presence + checkbox selection |
