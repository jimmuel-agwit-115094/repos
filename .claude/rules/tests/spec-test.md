# Angular Spec Tests (Karma / Jasmine)

Unit tests for Angular components, services, guards, and routing. Files colocated with source (`*.spec.ts`). Framework: Karma + Jasmine.

Reference: `CustomerServicePortal/src/WebSpa/ClientApp/src/app/**/*.spec.ts`

---

## Imports — NBS Test Helpers

```typescript
import { ComponentElements, ComponentFixture, DummyComponent } from '@nbs/ng-testing/karma';
import { NbsGuardTestHelper } from '@nbs/ng-xlr8-toolkit/testing';
import { NbsErrorPageComponent } from '@nbs/ng-xlr8-toolkit/error-page';
```

- `ComponentElements` — typed DOM query helper (replaces raw `nativeElement` queries)
- `ComponentFixture` — NBS wrapper around Angular's fixture with `whenStable()` and `create()`
- `DummyComponent` — placeholder for routes that don't need a real component
- `NbsGuardTestHelper` — for all `canMatch` / `canActivate` guard tests
- `NbsErrorPageComponent` — always include as the error route target (`error/:errorId`)

---

## 1. Guard Tests (`*.guard.spec.ts`, `*-routing.module.spec.ts`)

All guard tests use `NbsGuardTestHelper`. Same pattern for `canMatch` and `canActivate`.

```typescript
import { NbsGuardTestHelper } from '@nbs/ng-xlr8-toolkit/testing';
import { NbsErrorPageComponent } from '@nbs/ng-xlr8-toolkit/error-page';
import { DummyComponent } from '@nbs/ng-testing/karma';
import { SecurityService } from '../security/security.service';
import { securityMatchGuard } from './security.guard';

describe('People Search Route Guards', () => {
  let guardTestHelper: NbsGuardTestHelper;
  let mockCanUsePeopleSearch: boolean;  // outer scope — set in nested beforeEach

  beforeEach(async () => {
    const routes = [
      {
        path: 'people-search',
        component: DummyComponent,          // use DummyComponent unless testing the actual component
        canMatch: [securityMatchGuard],     // guard under test
        data: {
          securityCheck: (access: any) => !!access.canUsePeopleSearch
        }
      },
      {
        path: 'error/:errorId',             // REQUIRED — guard redirects here on deny
        component: NbsErrorPageComponent
      }
    ];

    const securityServiceMock = {
      access: () => ({ canUsePeopleSearch: mockCanUsePeopleSearch })
    } as any;

    guardTestHelper = new NbsGuardTestHelper();
    await guardTestHelper.setup(securityMatchGuard, routes, [
      { provide: SecurityService, useValue: securityServiceMock }
    ]);
  });

  describe('when user has permission', () => {
    beforeEach(() => { mockCanUsePeopleSearch = true; });

    it('should allow navigation', async () => {
      await guardTestHelper.navigateByUrl('people-search');
      expect(guardTestHelper.locationPath).toBe('/people-search');
    });
  });

  describe('when user does NOT have permission', () => {
    beforeEach(() => { mockCanUsePeopleSearch = false; });

    it('should redirect to /error/403', async () => {
      await guardTestHelper.navigateByUrl('people-search');
      expect(guardTestHelper.locationPath).toBe('/error/403');
    });
  });

  describe('edge cases', () => {
    beforeEach(() => { mockCanUsePeopleSearch = undefined as any; });

    it('should redirect when permission is undefined', async () => {
      await guardTestHelper.navigateByUrl('people-search');
      expect(guardTestHelper.locationPath).toBe('/error/403');
    });
  });
});
```

**Rules:**
- `guardTestHelper.setup(guard, routes, providers)` — always `await`
- `error/:errorId` route is mandatory in the routes array — the guard redirects there on deny
- Set permission in nested `beforeEach`, not in the outer one — outer `beforeEach` only sets up the helper
- Test `undefined` / falsy cases alongside `true` / `false`

**For `canActivate` guards** (class-based, not functional), use standard `TestBed` instead:

```typescript
// people-search.guard.spec.ts — canActivate section
const peopleSearchServiceSpy = jasmine.createSpyObj('PeopleSearchService', ['access']);

TestBed.configureTestingModule({
  providers: [
    PeopleSearchAuthGuard,
    { provide: PeopleSearchService, useValue: peopleSearchServiceSpy },
    { provide: Router, useValue: {
        createUrlTree: jasmine.createSpy('createUrlTree').and.returnValue(mockUrlTree)
    }}
  ]
});

guard = TestBed.inject(PeopleSearchAuthGuard);
peopleSearchService = TestBed.inject(PeopleSearchService) as jasmine.SpyObj<PeopleSearchService>;

// Assert redirect returns UrlTree
const result = guard.canActivate(mockRoute);
expect(result).toBe(mockUrlTree);
// eslint-disable-next-line @typescript-eslint/unbound-method
expect(router.createUrlTree).toHaveBeenCalledWith(['error/403']);
```

Note: `// eslint-disable-next-line @typescript-eslint/unbound-method` required before any `expect(spy.method).toHaveBeenCalled()`.

---

## 2. Component Tests (`*.component.spec.ts`)

### TestBed Setup

```typescript
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';
import { TestBed, fakeAsync, tick } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { NoopAnimationsModule } from '@angular/platform-browser/animations';
import { provideRouter } from '@angular/router';
import { ComponentElements, ComponentFixture } from '@nbs/ng-testing/karma';
import { PeopleSearchComponent } from './people-search.component';
import { PeopleSearchService } from './people-search.service';
import { TranslationService } from '../../shared/translation.service';

describe('PeopleSearchComponent', () => {
  let component: PeopleSearchComponent;
  let fixture: ComponentFixture<PeopleSearchComponent>;
  let page: PeopleSearchComponentElements;
  let mockService: jasmine.SpyObj<PeopleSearchService>;

  beforeEach(async () => {
    mockService = jasmine.createSpyObj('PeopleSearchService', ['search', 'getLastSearch', 'saveLastSearch']);
    // Set up default return values on the spy before configureTestingModule
    mockService.getLastSearch.and.returnValue(Promise.resolve(undefined));
    mockService.saveLastSearch.and.returnValue(Promise.resolve());
    mockService.search.and.returnValue(Promise.resolve({ resultCount: 0, people: [] }));

    await TestBed.configureTestingModule({
      declarations: [PeopleSearchComponent],   // NgModule components: always declarations
      imports: [
        FormsModule,                           // for ngModel
        NoopAnimationsModule,                  // suppress Material animation warnings
        // Material modules the template actually uses
        MatCardModule,
        MatFormFieldModule,
        MatInputModule,
        MatIconModule,
      ],
      providers: [
        provideRouter([{ path: 'test', component: MockPeopleSearchComponent }]),
        { provide: PeopleSearchService, useValue: mockService },
        { provide: TranslationService, useValue: mockTranslationService }
      ],
      schemas: [CUSTOM_ELEMENTS_SCHEMA]        // suppress unknown custom element warnings
    }).compileComponents();

    fixture = await ComponentFixture.create(PeopleSearchComponent);  // NBS wrapper
    page = new PeopleSearchComponentElements(fixture);
    component = page.component;
    await page.whenStable();
  });
```

**Key rules:**
- `declarations: [ComponentUnderTest]` — components are NgModule-based (`standalone: false`)
- `NoopAnimationsModule` — always import to suppress animation errors in Material
- `CUSTOM_ELEMENTS_SCHEMA` — suppresses custom element (`<npds-*>`, `<xlr8-*>`) warnings
- `provideRouter([])` — needed when component injects `Router` or `ActivatedRoute`
- All injected services: `{ provide: X, useValue: jasmine.createSpyObj(...) }`
- `ComponentFixture.create(Component)` — NBS wrapper, always `await`

### Mock Components

When a child component would pull in heavy dependencies, stub it:

```typescript
@Component({
  template: '<div>Mock</div>',
  standalone: false           // REQUIRED — matches NgModule pattern
})
class MockPeopleSearchComponent { }
```

### ComponentElements Pattern

Typed DOM query helper. Declare one class per component spec:

```typescript
class PeopleSearchComponentElements extends ComponentElements<PeopleSearchComponent> {
  // Use data-test-id selectors — matches E2E locators
  readonly heading       = this.getElement('Page Heading',   '[data-test-id="npds_heading_peopleSearch"]');
  readonly searchInput   = this.getElement('Search Input',   '[data-test-id="input_people_search"]');
  readonly searchButton  = this.getElement('Search Button',  '[data-test-id="npds_button_search"]');
  readonly searchForm    = this.getElement('Search Form',    '[data-test-id="form_peopleSearch"]');
  readonly warningIcon   = this.getElement('Warning Icon',   '[data-test-id="icon_warningBlankSearch"]');
  readonly progressBar   = this.getElement('Progress Bar',   '[data-test-id="progress-bar"]');
}
```

**`ComponentElements` API:**

```typescript
// Existence
expect(page.heading).toExist();

// Text content
expect(page.heading).textToMatch('People Search');

// Attribute
expect(page.searchInput).toHaveAttributeValue('placeholder', 'Search');

// CSS class
expect(page.searchFormField).toHaveClass('error-field');
expect(page.searchFormField).not.toHaveClass('error-field');

// Interaction
await page.searchButton.click();
await page.searchInput.setValue('John');

// Accessibility (axe)
await expectAsync(page).toBeAccessibleAsync();

// Wait
await page.whenStable();
```

### Async in Component Tests

**`fakeAsync` + `tick()`** — use when component method calls `Promise` internally (component is NOT `async`):

```typescript
it('should populate results after search', fakeAsync(() => {
  mockService.search.and.returnValue(Promise.resolve({
    people: [{ personId: 'p1', ... }],
    resultCount: 1
  }));

  component.searchText = 'John';
  component.onSearch();

  expect(component.isLoading).toBeTrue();  // synchronous check before tick

  tick();  // flush all pending promises

  expect(component.displayedResults.length).toBe(1);
  expect(component.isLoading).toBeFalse();
}));
```

**`async/await`** — use in `beforeEach` and when testing `async` component methods directly:

```typescript
it('should update model on input', async () => {
  await page.searchInput.setValue('test');
  expect(component.searchText).toBe('test');
});
```

**Error handling — `spyOn(console, 'error')`:**

```typescript
it('should handle search errors gracefully', fakeAsync(() => {
  const consoleSpy = spyOn(console, 'error');
  mockService.search.and.returnValue(Promise.reject(new Error('API Error')));

  component.searchText = 'John';
  component.onSearch();
  tick();

  expect(consoleSpy).toHaveBeenCalledWith('Search error:', jasmine.any(Error));
  expect(component.isLoading).toBeFalse();
}));
```

**Multiple ticks for chained promises:**

```typescript
component.onSearch();
tick();  // resolves search promise
tick();  // resolves saveLastSearch promise that chains off it
```

### Spy Patterns

```typescript
// SpyObj — all methods are spies
mockService = jasmine.createSpyObj('PeopleSearchService', ['search', 'getLastSearch']);

// SpyObj with default return values in one call
mockTranslationService = jasmine.createSpyObj('TranslationService', {
  getPeopleSearchResultsMessage: '1 result for "John"',
  getPeopleSearchShowingResultsMessage: 'Showing 4 of 10'
});

// Spy on existing component method
const onSearchSpy = spyOn(component, 'onSearch').and.callThrough();

// Spy on global/DOM APIs
spyOn(document, 'getElementById').and.returnValue(mockInputElement);
spyOn(console, 'error');
spyOn(console, 'warn');

// Spy return values
mockService.search.and.returnValue(Promise.resolve({ people: [], resultCount: 0 }));
mockService.search.and.returnValue(Promise.reject(new Error('fail')));

// Capture spy call args
const searchArgs = mockService.search.calls.first().args[0];
expect(searchArgs).toEqual({ keyword: 'John', limit: 10, skip: 4 });

// eslint-disable-next-line required before these
expect(mockService.search).toHaveBeenCalledWith({ keyword: 'John', limit: 10, skip: 0 });
expect(mockService.saveLastSearch).not.toHaveBeenCalled();
```

---

## 3. Service Tests (`*.service.spec.ts`)

### HTTP Testing — Preferred Pattern (newer files)

```typescript
import { provideHttpClient, withInterceptorsFromDi } from '@angular/common/http';
import { provideHttpClientTesting, HttpTestingController } from '@angular/common/http/testing';

describe('TenantService', () => {
  let httpMock: HttpTestingController;

  beforeEach(() =>
    TestBed.configureTestingModule({
      providers: [
        provideHttpClient(withInterceptorsFromDi()),
        provideHttpClientTesting()
      ]
    }).compileComponents()
  );

  beforeEach(() => {
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();  // ALWAYS — ensures no unexpected HTTP calls
  });
```

### HTTP Testing — Legacy Pattern (still present in some files)

```typescript
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

TestBed.configureTestingModule({
  imports: [HttpClientTestingModule],
  providers: [ServiceUnderTest]
});
```

Both patterns coexist in the codebase. Prefer `provideHttpClient` for new tests.

### HTTP Mock Pattern

```typescript
it('should POST to correct endpoint', async () => {
  const searchPromise = service.search({ keyword: 'John', limit: 10 });

  const req = httpMock.expectOne('api/people-search');
  expect(req.request.method).toBe('POST');
  expect(req.request.body).toEqual({ keyword: 'John', limit: 10, skip: 0 });

  req.flush(mockResponse);         // resolve with success

  const result = await searchPromise;
  expect(result).toEqual(mockResponse);
});

// Error responses
req.flush('Server Error', { status: 500, statusText: 'Internal Server Error' });
await expectAsync(searchPromise).toBeRejected();

// Network error
req.error(new ProgressEvent('Network error'), { status: 0 });
await expectAsync(searchPromise).toBeRejected();
```

### Service Mock File (`*.service.mock.ts`)

When HTTP setup is complex, extract to a sibling mock file:

```typescript
// tenant.service.mock.ts
export function matchGetCurrentContextTenant(
  httpMock: HttpTestingController,
  tenantId: string,
  tree: TenantTree[]
): void {
  const req = httpMock.expectOne('api/tenant/tenant-context');
  req.flush({ CurrentContextTenantId: tenantId, TenantTree: tree });
}
```

Used in tests as: `matchGetCurrentContextTenant(httpMock, tenantId, tree)`

### Observable Caching Test

```typescript
it('should cache responses (ReplaySubject)', inject([TenantService], async (sut: TenantService) => {
  const obs$ = sut.getCurrentContextTenant();
  matchGetCurrentContextTenant(httpMock, tenantId, tree);

  await obs$.toPromise();

  // Second call — no new HTTP request (cached)
  sut.getCurrentContextTenant();
  expect(httpMock.verify.bind(httpMock)).not.toThrow();
}));

it('should NOT cache errors', inject([TenantService], async (sut: TenantService) => {
  const obs$ = sut.getCurrentContextTenant();
  matchGetCurrentContextTenantError(httpMock, 'error');

  await expectAsync(obs$.toPromise()).toBeRejected();

  // After error, next call retries HTTP
  sut.getCurrentContextTenant();
  matchGetCurrentContextTenant(httpMock, tenantId, tree);  // must match a new request
}));
```

### `inject()` Helper (alternative to `TestBed.inject`)

```typescript
it('should cache', inject([TenantService], async (sut: TenantService) => {
  // sut is injected directly into the test function
}));
```

---

## 4. Accessibility

```typescript
it('should be accessible', async () => {
  await expectAsync(page).toBeAccessibleAsync();
});
```

- Provided by `@nbs/ng-testing/karma`
- Call after page is stable (after `beforeEach` setup)
- One accessibility test per component spec is standard

---

## 5. Test Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Component spec | `{ComponentName}.spec.ts` | `people-search.component.spec.ts` |
| Service spec | `{ServiceName}.spec.ts` | `people-search.service.spec.ts` |
| Guard spec | `{feature}.guard.spec.ts` | `people-search.guard.spec.ts` |
| Routing spec | `{feature}-routing.module.spec.ts` | `people-search-routing.module.spec.ts` |
| Mock helpers | `{ServiceName}.service.mock.ts` | `tenant.service.mock.ts` |

**`describe` / `it` naming:**
- Top `describe`: class/file name (e.g., `'People Search Component'`)
- Nested `describe`: method name or scenario (e.g., `'clearSearch'`, `'when user has permission'`)
- `it`: `'should {expected behavior}'`

---

## 6. Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Missing `standalone: false` on mock components | Always set it — all CSP components use NgModule pattern |
| `unbound-method` lint error on spy assertions | Add `// eslint-disable-next-line @typescript-eslint/unbound-method` before the `expect` |
| `httpMock.verify()` missing | Add to `afterEach` — fails silently without it |
| Forgetting `tick()` for chained promises | Count promise chains — each `.then()` / chained `await` needs its own `tick()` |
| Angular Material animation errors | Import `NoopAnimationsModule` |
| Custom element warnings | Add `CUSTOM_ELEMENTS_SCHEMA` to TestBed |
| `provideRouter` missing for routed components | Always include even if routing isn't directly tested |
| `NbsGuardTestHelper` — missing error route | `error/:errorId` with `NbsErrorPageComponent` is mandatory in routes array |
| Setting mock state in outer `beforeEach` | Set permission booleans in nested `beforeEach` — outer sets up the helper only |
