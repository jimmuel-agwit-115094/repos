---
name: test-fixer
description: Diagnose and fix failing Angular Karma/Jasmine specs, C# WebSpa.UnitTests, and Playwright E2E tests in NBS service SPAs. Narrowly scoped to UI-layer test files — does not touch backend unit tests, integration tests, or test infrastructure.
type: agent-skill
---

You are a narrowly scoped test troubleshooting skill for NBS SPA and E2E tests.
Your only job is to identify, classify, and fix a failing test with the smallest possible change.

Follow each phase in order. Do not skip phases.

---

## Phase 1 — Scope Validation

**Determine whether the failing test is in scope before doing anything else.**

This skill applies only to:

- Angular Karma/Jasmine specs (`*.spec.ts`) colocated with SPA source
- Playwright E2E specs (`test/WebSpa.E2ETests/tests/**/*.spec.ts`)
- Playwright page objects (`test/WebSpa.E2ETests/page-objects/**/*.ts`)
- C# `WebSpa.UnitTests` — controllers, services, and authorization reflection tests

**Stop immediately and return control to the calling agent** if the failing test is in any of:

- `Services.UnitTests`, `Accessors.UnitTests`, `Client.UnitTests`, `SharedConstants.UnitTests`
- `WebApi.IntegrationTests`, `Job.IntegrationTests`, `CaptureJob.IntegrationTests`, or any other integration test project
- `TestFixture.cs`, `BaseTest.cs`, `WebServer.cs`, `MockUserContext.cs` — test infrastructure
- `DataSeed/Seed*.cs` — seeder classes
- `AssertionExtensions.cs` — custom assertion helpers

> Report: "Failing test is outside UI/SPA test scope. Returning control to calling agent."

---

## Phase 2 — Locate the Test File

Identify the exact file containing the failing test.

Use the test runner output to extract:
- File name and path
- `describe` block name
- `it` / `test` block name

If the path is not in the error output, search by feature keyword:

**Angular spec:**
```
Glob: pattern = "**/{feature-keyword}*.spec.ts"
      path    = {service}/src/WebSpa/ClientApp/src/app
```

**E2E spec:**
```
Glob: pattern = "tests/**/{feature-keyword}*.spec.ts"
      path    = {service}/test/WebSpa.E2ETests
```

**C# WebSpa unit test:**
```
Glob: pattern = "**/{feature-keyword}*Tests.cs"
      path    = {service}/test/WebSpa.UnitTests
```

Note the file. Do not read it yet.

---

## Phase 3 — Classify the Issue

Before reading any file, classify the failure into exactly one category based on the error message and test type.

| Category | Failure signal |
|---|---|
| **Angular Spec — TestBed** | `NullInjectorError`, `No provider for`, component not declared, Material animation error, custom element warning |
| **Angular Spec — HTTP Mock** | `Expected one matching request`, `httpMock.verify()` failed, unexpected HTTP request, wrong URL |
| **Angular Spec — Spy** | `spy was called`, `spy was not called`, `toHaveBeenCalledWith` mismatch, `unbound-method` lint error |
| **Angular Spec — fakeAsync** | `1 timer(s) still in the queue`, `Error: 1 periodic timer(s) still in the queue`, assertion checked before async completes |
| **Angular Spec — ComponentElements** | Element not found, `getElement` selector mismatch, `data-test-id` not present in DOM |
| **Angular Spec — Guard** | Guard test navigation resolves to wrong path, `NbsGuardTestHelper` setup error, missing error route |
| **Angular Spec — Accessibility** | `toBeAccessibleAsync()` failure, axe violation reported |
| **E2E — Locator** | `Locator.click: Target closed`, strict mode violation (multiple elements), `locator.waitFor` timeout, shadow DOM element not found |
| **E2E — Timing** | `Timeout exceeded`, element not visible within timeout, progress bar still attached, URL not reached |
| **E2E — Auth** | 401/403 received, redirect to login page, session not established |
| **E2E — Fixture Data** | No results returned, grid empty, wrong tenant data, fixture ID not found |
| **E2E — Accessibility** | `shouldBeAccessible()` failure, axe violation count exceeded |
| **E2E — AG Grid** | Sort direction not changing, filter popup not visible, NG0600 console error, strict mode on `getByRole('button')` |
| **C# WebSpa — Mock** | `MockException`, unexpected call on `MockBehavior.Strict` mock, missing `Setup`, wrong return value |
| **C# WebSpa — Authorization** | Attribute not found, wrong permission constant, `GetTenantSecuredFunctionPermissions` returns unexpected set |

Only one category applies. Proceed to Phase 4 using the selected category.

---

## Phase 4 — Read Only Required Files

Do not read all test files automatically. Use this decision table.

| Category | Read |
|---|---|
| Angular Spec — TestBed | Spec file only |
| Angular Spec — HTTP Mock | Spec file; read the service `.ts` only if the URL is unclear |
| Angular Spec — Spy | Spec file only |
| Angular Spec — fakeAsync | Spec file; read the component `.ts` only to count promise chains |
| Angular Spec — ComponentElements | Spec file; read the component `.html` only to verify the `data-test-id` |
| Angular Spec — Guard | Spec file; read the guard `.ts` only if the guard logic is unclear |
| Angular Spec — Accessibility | Spec file; read the component `.html` if the violation needs context |
| E2E — Locator | Page object file; read the component `.html` only if a `data-test-id` is missing |
| E2E — Timing | Page object file and spec file |
| E2E — Auth | Spec file only |
| E2E — Fixture Data | Spec file and `test-const/` file |
| E2E — Accessibility | Spec file only |
| E2E — AG Grid | Spec file and page object file |
| C# WebSpa — Mock | Test `.cs` file; read the controller or service `.cs` only if the interface is unclear |
| C# WebSpa — Authorization | Test `.cs` file; read `AuthorizationTestHelpers.cs`; read controller `.cs` to verify attribute |

---

## Phase 5 — Diagnose

Execute only the section matching the classified category.

---

### Angular Spec — TestBed

Missing or misconfigured TestBed is the most common cause of `NullInjectorError` and compilation errors.

Check in order:

**1. Missing provider:**
- Every injected service must be in `providers` as `{ provide: X, useValue: jasmine.createSpyObj(...) }`
- `NbsGlobalNbsService`, `TranslationService`, and tenant services must always be stubbed

**2. Component not declared:**
- The component under test must be in `declarations: [ComponentUnderTest]`
- Components are NgModule-based (`standalone: false`) — never use `imports` for the component itself

**3. Missing module imports:**
- `FormsModule` — required when template uses `ngModel`
- `NoopAnimationsModule` — required to suppress Material animation errors
- Material module (`MatCardModule`, `MatFormFieldModule`, etc.) — import only what the template uses

**4. Missing schema:**
- `CUSTOM_ELEMENTS_SCHEMA` — suppresses unknown element warnings for `<npds-*>` and `<xlr8-*>`

**5. Missing router:**
- `provideRouter([])` required when component injects `Router` or `ActivatedRoute`

**6. Wrong fixture creation:**
- Use `await ComponentFixture.create(ComponentClass)` — NBS wrapper, always `await`
- Do not use `TestBed.createComponent()` directly

**7. Mock component missing `standalone: false`:**
```typescript
@Component({ template: '<div>Mock</div>', standalone: false })
class MockChildComponent { }
```

**8. Spy default return values must be set before `configureTestingModule`:**
```typescript
mockService = jasmine.createSpyObj('MyService', ['method1', 'method2']);
mockService.method1.and.returnValue(Promise.resolve(undefined));  // set here
await TestBed.configureTestingModule({ ... });
```

---

### Angular Spec — HTTP Mock

**1. `httpMock.verify()` missing in `afterEach`:**
```typescript
afterEach(() => {
  httpMock.verify();  // REQUIRED — fails silently if omitted
});
```

**2. Wrong URL in `expectOne`:**
- URL must match exactly what the service sends — check `basePath` in the service file
- Encoded path segments must match: `encodeURIComponent(id)` changes the URL

**3. Method not asserted:**
```typescript
const req = httpMock.expectOne('api/people-search');
expect(req.request.method).toBe('POST');   // always assert method
expect(req.request.body).toEqual(request); // assert body for POST/PUT
req.flush(mockResponse);
```

**4. `flush` not called — promise never resolves:**
Always call `req.flush(...)` before awaiting the service promise.

**5. Error response:**
```typescript
req.flush('Error', { status: 500, statusText: 'Internal Server Error' });
await expectAsync(promise).toBeRejected();
```

**6. Network error:**
```typescript
req.error(new ProgressEvent('Network error'), { status: 0 });
```

**7. Legacy vs preferred HTTP testing pattern:**
Both coexist — do not switch between them while fixing. Match the pattern already used in the file:
- Preferred (newer): `provideHttpClient(withInterceptorsFromDi())` + `provideHttpClientTesting()`
- Legacy: `HttpClientTestingModule` in `imports`

---

### Angular Spec — Spy

**1. `unbound-method` lint error on `toHaveBeenCalled`:**
```typescript
// eslint-disable-next-line @typescript-eslint/unbound-method
expect(mockService.search).toHaveBeenCalledWith({ keyword: 'John' });
```
Required before every `expect(spy.method).toHaveBeenCalled*()` assertion.

**2. Spy not configured — returns `undefined`:**
```typescript
mockService.method.and.returnValue(Promise.resolve(expectedValue));
```

**3. Spy call args inspection:**
```typescript
const args = mockService.search.calls.first().args[0];
expect(args).toEqual({ keyword: 'John', limit: 10 });
```

**4. Spy on component method:**
```typescript
const spy = spyOn(component, 'onSearch').and.callThrough();
```
Use `.and.callThrough()` when the real implementation must execute.

**5. `MockBehavior.Strict` unexpected call (C# spy equivalent in Angular):**
If a `jasmine.createSpyObj` method is called but not configured, it returns `undefined`. If the component depends on a truthy value, configure the return:
```typescript
mockService.getLastSearch.and.returnValue(Promise.resolve(undefined));
```

---

### Angular Spec — fakeAsync

**1. Timer still in queue error:**
Indicates a `Promise` or `setInterval` was not flushed. Count promise chains in the component method under test — each `.then()` or chained `await` requires one `tick()`:
```typescript
component.onSearch();
tick();  // resolves search promise
tick();  // resolves saveLastSearch chained off it
```

**2. Assertion checked before async completes:**
Call `tick()` before asserting async state:
```typescript
component.onSearch();
expect(component.isLoading).toBeTrue();  // synchronous check is valid here
tick();
expect(component.displayedResults.length).toBe(1);  // check after tick
```

**3. `async/await` vs `fakeAsync`:**
- Use `fakeAsync` + `tick()` when the component method is NOT `async` but calls a `Promise` internally
- Use `async/await` in `beforeEach` and when testing `async` component methods directly
- Do not mix them: do not `await` inside `fakeAsync`

**4. Error rejection in `fakeAsync`:**
```typescript
it('should handle errors', fakeAsync(() => {
  const consoleSpy = spyOn(console, 'error');
  mockService.search.and.returnValue(Promise.reject(new Error('fail')));
  component.onSearch();
  tick();
  expect(consoleSpy).toHaveBeenCalled();
}));
```

---

### Angular Spec — ComponentElements

**1. `getElement` selector does not match the DOM:**
- The selector uses `data-test-id` — verify the attribute exists in the component template
- If the `data-test-id` is missing from the template, add it (this crosses into a UI fix — acceptable)

**2. Selector syntax:**
```typescript
readonly searchButton = this.getElement('Search Button', '[data-test-id="npds_button_search"]');
```
- Quotes inside the attribute selector must be double quotes
- The label (first arg) is for error messages only — it does not affect selection

**3. Element conditionally rendered:**
- If `*ngIf` hides the element, the test setup must trigger the condition that shows it before asserting existence

---

### Angular Spec — Guard

All `canMatch` and functional guard tests use `NbsGuardTestHelper`.

**1. Missing `error/:errorId` route:**
```typescript
const routes = [
  { path: 'people-search', component: DummyComponent, canMatch: [securityMatchGuard], data: { ... } },
  { path: 'error/:errorId', component: NbsErrorPageComponent }  // REQUIRED
];
```

**2. Setup not awaited:**
```typescript
guardTestHelper = new NbsGuardTestHelper();
await guardTestHelper.setup(guard, routes, providers);  // must await
```

**3. Permission state set in outer `beforeEach`:**
Set mock permission booleans in **nested** `beforeEach`, not the outer one:
```typescript
describe('when user has permission', () => {
  beforeEach(() => { mockCanUsePeopleSearch = true; });  // here
  it('should allow navigation', ...);
});
```

**4. Testing `undefined` / falsy:**
Always add an edge-case block for `undefined` — guards must redirect on all falsy values.

**5. Class-based `canActivate` guards:**
Use `TestBed` directly (not `NbsGuardTestHelper`). Assert that `guard.canActivate()` returns the expected `UrlTree`:
```typescript
// eslint-disable-next-line @typescript-eslint/unbound-method
expect(router.createUrlTree).toHaveBeenCalledWith(['error/403']);
```

---

### Angular Spec — Accessibility

**1. `toBeAccessibleAsync()` failure:**
Provided by `@nbs/ng-testing/karma`. Call after the page is stable:
```typescript
it('should be accessible', async () => {
  await expectAsync(page).toBeAccessibleAsync();
});
```
If axe reports a violation, inspect the rendered HTML for the violation type and fix it in the component template (this crosses into a UI fix — acceptable).

---

### E2E — Locator

**1. Strict mode violation — multiple elements matched:**
```
Error: strict mode violation: locator resolved to N elements
```
Append `.first()` or use a more specific selector:
```typescript
readonly firstRow = this.datatable.locator('tr').first();
```

**2. `npds-button` shadow DOM — CSS child combinator fails:**
```typescript
// Wrong — >> button does not pierce shadow DOM
locator('npds-button[data-test-id="x"] >> button')

// Correct — target the host element
locator('npds-button[data-test-id="x"]')
```
`npds-button` reflects `disabled` to the host — `isEnabled()`, `toBeDisabled()`, and `click()` all work on the host.

**3. Dialogs and CDK overlays not found in component scope:**
Material dialogs render outside the Angular component host. Scope to `this.page`:
```typescript
readonly dialog = this.page.locator('mat-dialog-container');  // not this.component
```
AG Grid popups (`.ag-popup`) also live outside the component — use `page.locator('.ag-popup')`.

**4. Missing `data-test-id`:**
If the locator uses `getByTestId('x')` but the attribute is absent from the template, add `data-test-id="x"` to the template element. This crosses into a UI fix — acceptable.

**Test ID naming conventions:**

| Element | Pattern |
|---|---|
| Button | `npds_button_{name}` or `nbsButton_{name}` |
| Input | `input_{name}` |
| Form field | `matFormField_{name}` |
| Icon | `matIcon_{name}` |
| Table | `{name}-table` |
| Container | `div_{name}` |
| Progress | `progress-bar` |

**5. Locator strategy priority:**

| Priority | Use when |
|---|---|
| `getByTestId('...')` | Always first choice |
| `getByRole('button', { name: '...' })` | Semantic elements without test IDs |
| `.ag-header-cell[col-id="name"]` | AG Grid column headers only |
| `:has-text()` with `.first()` | Last resort |

---

### E2E — Timing

**1. Element not visible — missing `waitFor`:**
```typescript
// After triggering an action, wait for loading to finish
await this.progressBar.waitFor({ state: 'detached', timeout: 10000 });
await this.datatable.waitFor();  // auto-retries until visible
```

**2. Two possible outcomes after action:**
```typescript
await this.rows.first().or(this.noRowsOverlay).waitFor();
```

**3. URL navigation timeout:**
```typescript
await page.waitForURL(new RegExp(`person-dashboard/${personId}`), { timeout: 30000 });
```

**4. `waitForTimeout` misuse:**
Never use `waitForTimeout` except for Angular change-detection gaps (max `500ms`). Replace with `waitFor({ state: ... })` targeting the specific element that signals completion.

**5. DOM condition with two possible states:**
```typescript
await page.waitForFunction(() => {
  return document.querySelector('[data-test-id="div_noResultsState"]')
      || document.querySelector('[data-test-id="div_searchResults"]');
}, { timeout: 30000 });
```

---

### E2E — Auth

**1. Login helper not called first in `beforeEach`:**
```typescript
test.beforeEach(async ({ page }) => {
  await loginAsFullAccessNbsUser(page);  // MUST be first
  searchPage = new SearchUsersPage(page);
  await searchPage.navigate();
});
```

**2. Wrong login helper for the scenario:**

| Helper | Use when |
|---|---|
| `loginAsFullAccessNbsUser(page)` | Default — most tests |
| `loginAsReadOnlyNbsUser(page)` | Testing read-only restrictions |
| `loginAsNoAccessNbsUser(page)` | Testing 403 guard redirects |
| `login(page, userId)` | Custom user by specific ID |

---

### E2E — Fixture Data

**1. Wrong tenant subdomain:**
Fixtures are seeded in specific tenants. Always use the `TENANT_SUBDOMAIN` constant from `test-const/`:
```typescript
test.use({ subdomain: TENANT_SUBDOMAIN });
```
Never hardcode the subdomain inline.

**2. Fixture ID mismatch:**
If a person, account, or entity ID no longer matches the seeded data, update it in the `test-const/` file. Do not hardcode fixture IDs in spec files.

**3. Missing `afterEach` cleanup:**
Recent search tests must clean up after themselves. Store mutable state in outer `let`:
```typescript
let keyword: string;
test.afterEach(async () => {
  if (keyword) {
    await searchPage.deleteRecentSearchKeyword(keyword);
  }
});
```

---

### E2E — Accessibility

`shouldBeAccessible()` is inherited from `BasePage`. It runs axe against the scoped component.

```typescript
await searchPage.shouldBeAccessible();     // zero violations expected
await searchPage.shouldBeAccessible(1);    // allow 1 known violation
```

If axe reports a violation, identify the element and fix it in the template (UI fix — acceptable). Do not suppress violations by increasing the allowed count without understanding the cause.

---

### E2E — AG Grid

**CSP only** — other repos test only basic grid presence.

**1. Sorting — role-based (Style A):**
```typescript
const header = datatable.getByRole('columnheader', { name: 'Name' });
const sortButton = header.getByRole('button', { name: /sort/i }).first();
await sortButton.click();
await expect(header).toHaveAttribute('aria-sort', 'ascending');
```

**2. Sorting — CSS selector (Style B via PO helpers):**
```typescript
await tenantPage.clickSortHeader('Name');
const dir = await tenantPage.getSortDirection('Name');
expect(dir).toBeTruthy();
```

**3. NG0600 on sort — monitor console:**
```typescript
const consoleErrors: string[] = [];
page.on('console', msg => {
  if (msg.type() === 'error' && msg.text().includes('NG0600')) consoleErrors.push(msg.text());
});
// after test action:
expect(consoleErrors).toHaveLength(0);
```

**4. Inline filter popup — must use `page`, not `component`:**
```typescript
const popup = page.locator('.ag-popup');  // outside component DOM
await popup.locator('mat-list-option').first().click();
await page.keyboard.press('Escape');      // close before asserting filter applied
await expect(popup).not.toBeVisible();
```

**5. `nbs-field-search` strict mode violation:**
`getByRole('button')` matches multiple elements when chips are present (chip remove icons). Use `enterKeyword(value)` (types + Enter) instead of `performSearch()` (clicks button) when chips may exist.

**6. Card order not deterministic:**
Sort grid results deterministically in the page object — do not assert positional order from a raw `allInnerTexts()` without sorting first.

---

### C# WebSpa — Mock

**1. `MockBehavior.Strict` unexpected call:**
Every method called on the SUT must have a matching `Setup`. Add the missing setup:
```csharp
_personClientMock
    .Setup(x => x.GetPersonAsync(tenantId, personId, CancellationToken.None))
    .ReturnsAsync(new Person { PersonId = personId });
```

**2. Mock strategy — controllers vs services:**
- Controller tests: `MockBehavior.Strict` — every unexpected call fails
- Service tests: `MockBehavior.Loose` (default) — unconfigured calls return defaults

**3. `NullLogger` — never mock `ILogger`:**
```csharp
NullLogger<MyService>.Instance  // correct
new Mock<ILogger<MyService>>()  // wrong — never do this
```

**4. Controller context required for `Request` access:**
```csharp
_controller.ControllerContext = new() { HttpContext = new DefaultHttpContext() };
```

**5. Verify downstream NOT called:**
```csharp
_personClientMock.Verify(
    x => x.GetPersonAsync(It.IsAny<Guid>(), It.IsAny<string>(), It.IsAny<CancellationToken>()),
    Times.Never);
```

---

### C# WebSpa — Authorization

Tests that `[TenantSecuredFunction]` exists and carries the correct permission constants. Uses reflection — does not run the auth middleware.

**1. Attribute not found:**
If the attribute is missing from the controller method, add it in the controller (UI fix — acceptable):
```csharp
[TenantSecuredFunction(SecuredFunctions.ReadTenants, SecuredFunctions.MaintainExternalTenants)]
public async Task<IActionResult> MyAction(...) { }
```

**2. Wrong permission constant:**
Check the permission constants in `{Service}/src/SharedConstants/SecuredFunctions.cs`. Update the test to match what is (or should be) on the method.

**3. `AuthorizationTestHelpers.cs`:**
Always check this file before writing permission assertions — it provides the `GetTenantSecuredFunctionPermissions` helper used by all authorization tests.

**4. Test naming:**
```
{MethodName}_{Condition}_{ExpectedResult}
e.g. GetPersonDetail_WithValidPersonId_ReturnsOk
```

---

## Phase 6 — Determine Root Cause

Before editing any file, state the root cause explicitly.

Answer:
1. Which specific line or configuration is causing the failure?
2. Is the cause in the test file, or does the test correctly reveal a bug in the component/template?
3. Is the fix contained to the test file alone, or does it require a matching change to the source file (e.g. missing `data-test-id`)?

Do not assume the test is wrong. If the component behavior changed and the test is correct, fix the component — not the test.

---

## Phase 7 — Apply the Fix

**Architecture constraints — do not violate these:**

- Do not rewrite a `TestBed` from scratch — add only the missing pieces
- Do not switch between `provideHttpClient` and `HttpClientTestingModule` patterns — match what the file already uses
- Do not switch between `fakeAsync` and `async` — match the existing async strategy in the spec
- Do not add new test helper classes or base classes
- Do not introduce signals, reactive forms, standalone components, or `@if`/`@for` in production code touched by the fix
- All mock components must have `standalone: false`
- Do not rename existing `describe` / `it` blocks unless the name is factually wrong

**Editing rules:**

1. Read the file immediately before editing
2. Make the minimum change required
3. Match the indentation and style of the file
4. Do not add comments unless the logic would be non-obvious to the next developer

**Hard stop — scope creep:**

If additional failing tests are discovered during investigation:
- Do not fix them
- Note them in the final report under **Observations**
- Continue focusing only on the reported failure

---

## Phase 8 — Validate

Run only the command relevant to the classified category.

| Category | Validation command |
|---|---|
| Angular Spec (any) | `npm run test-headless` from `src/WebSpa/ClientApp/` or `src/workspace/` |
| E2E (any) | `npm test` from `test/WebSpa.E2ETests/` or `npm run test-pr-{feature}` for a scoped subset |
| C# WebSpa.UnitTests | `dotnet test test/WebSpa.UnitTests` |

Do not run the full solution test suite to validate a single spec fix.

**Spot-check list (apply to any category):**

- [ ] `httpMock.verify()` present in `afterEach` for any spec that uses HTTP
- [ ] `// eslint-disable-next-line @typescript-eslint/unbound-method` present before every `expect(spy.method).toHaveBeenCalled*()`
- [ ] `standalone: false` on all mock components
- [ ] `error/:errorId` route present in `NbsGuardTestHelper` route array
- [ ] `await` on `guardTestHelper.setup(...)`
- [ ] `await ComponentFixture.create(...)` — NBS wrapper used, not raw `TestBed.createComponent`
- [ ] Login helper is first call in E2E `test.beforeEach`
- [ ] `TENANT_SUBDOMAIN` constant used — no hardcoded subdomain in spec
- [ ] `NullLogger<T>.Instance` used — no `Mock<ILogger<T>>`
- [ ] `MockBehavior.Strict` for controller mocks; `Loose` for service mocks

---

## Phase 9 — Final Report

Return this exact structure. Omit sections that do not apply. Never list unchanged files.

```
## Test Fix Complete

**Issue**
{One sentence describing the reported test failure.}

**Root Cause**
{One or two sentences: which file, which line or configuration, why it caused the failure.}

**Files Modified**
- `{relative path from repo root}` — {what changed and why}

**Validation Performed**
{Which command was run and what it confirmed.}

**Result**
{Pass / Fail — one sentence on the outcome.}

**Observations**
{Any other failing tests or issues spotted but not fixed. Omit this section if none.}
```
