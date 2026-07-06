# Frontend (Angular) — CustomerServicePortal

Root: `CustomerServicePortal/src/WebSpa/ClientApp/src/app/`

---

## Module Structure

```text
src/app/
├── app.module.ts                    # Root module, interceptors, locale registration
├── app-routing.module.ts            # Top-level lazy routes + guards
├── app.component.ts                 # Menu setup, Pendo init, theme
├── search/                          # User & tenant search
│   ├── search-tab/                  # Tab-based search (user/tenant toggle)
│   │   └── user-search/             # User keyword + advanced search
│   └── search-tenants/              # Tenant search
├── people-management/               # People search & person detail
│   ├── people-search/               # People search + person-card child
│   ├── person-dashboard/            # Individual person view + account-card child
│   └── person-detail/               # Overlay detail + password-reset dialog
├── tenants/                         # Tenant management
└── shared/                          # Guards, interceptors, services, utilities
    ├── shared.module.ts             # Central Material + NBS re-export module
    ├── trace.service.ts             # Datadog RUM + logs
    ├── translation.service.ts       # i18n via $localize
    ├── security/                    # SecurityService + route guards
    ├── interceptors/                # session-expired.interceptor
    ├── featureflag/                 # FeatureFlagService (LaunchDarkly)
    ├── date-functions/              # Date utilities
    └── timezone/                    # Timezone service
```

---

## Component Pattern

**All components use NgModules — not standalone.**  
Set `standalone: false` explicitly on every component.

### Standard file layout per feature

```text
feature-name/
├── feature-name.component.ts
├── feature-name.component.html
├── feature-name.component.scss
├── feature-name.module.ts           # NgModule with declarations/imports
├── feature-name-routing.module.ts   # Routes + canMatch/canActivate + data
├── feature-name.service.ts
├── feature-name.interface.ts
├── feature-name.enum.ts
└── child-component/
    ├── child-component.component.ts
    ├── child-component.component.html
    └── child-component.component.scss
```

### Dependency Injection

Use `inject()` — not constructor injection:

```typescript
@Injectable({ providedIn: 'root' })
export class MyService {
  private readonly http = inject(HttpClient);
  private readonly traceService = inject(TraceService);
}
```

### Change Detection

- Container/page components: default `ChangeDetectionStrategy`
- Leaf/presentational components: `ChangeDetectionStrategy.OnPush`
- When using `OnPush` with `@Input` setter, call `cdr.markForCheck()` after state update:

```typescript
// person-card.component.ts
@Input()
get personSource(): PersonSearchResult { return this._person; }
set personSource(value: PersonSearchResult) {
  this._person = value;
  this.formatDisplayFields();
  this.cdr.markForCheck();
}
```

---

## Service Pattern

### HTTP calls — `lastValueFrom` + Promise

All services convert observables to Promises. Components use `async/await`.

```typescript
@Injectable({ providedIn: 'root' })
export class PeopleSearchService {
  private readonly http = inject(HttpClient);
  readonly basePath = 'api/people-search';

  search(request: PeopleSearchRequest): Promise<PeopleSearchResult> {
    return lastValueFrom(
      this.http.post<PeopleSearchResult>(this.basePath, request)
    );
  }
}
```

### Error handling

Services let errors bubble. Components handle with `.catch()` or `try/catch`:

```typescript
// people-search.component.ts
this.peopleSearchService.search(request)
  .then(result => { this.displayedResults = result.people; })
  .catch((error: unknown) => {
    this.traceService.logHttpError('People search failed', error as NbsError, { keyword });
  })
  .finally(() => { this.isLoading = false; });
```

**Exception:** services that need silent degradation use `catchError` internally (e.g. `UserRecentSearchService.all()` returns `null` on failure).

### TraceService (Datadog)

```typescript
// HTTP errors — structured
this.traceService.logHttpError('Failed to load dashboard', error, { personId });

// General errors
this.traceService.logError('Unexpected error in search', error, { context: 'user-search' });

// User actions (RUM)
this.traceService.logUserAction('people-search-submitted', { keyword, resultCount });
```

### `access()` pattern

Services that are feature-gated expose an `access()` method used by route guards:

```typescript
access(): boolean {
  return this.tenantInfoService.isInternalTenant();
}
```

---

## State / Data Flow

No NgRx. State lives on component class properties.

### Component state
```typescript
export class PeopleSearchComponent implements AfterViewInit {
  searchText = '';
  displayedResults: PersonSearchResult[] = [];
  isLoading = false;
  showNoResultsMessage = false;
}
```

### Observable shared state — `ReplaySubject`

For services that cache async data:

```typescript
// tenant.service.ts
private tenantTree$?: ReplaySubject<CurrentContextTenant>;

getCurrentContextTenant() {
  if (!this.tenantTree$) {
    this.tenantTree$ = new ReplaySubject<CurrentContextTenant>(1);
    this.http.get<...>(...).pipe(
      catchError(err => {
        this.tenantTree$ = undefined; // reset so next call retries
        return throwError(() => err);
      })
    ).subscribe(this.tenantTree$);
  }
  return this.tenantTree$;
}
```

### @Input / @Output

```typescript
// Child emits event to parent
@Output() detailsClick = new EventEmitter<DetailsClickEvent>();

goToDetails(): void {
  this.detailsClick.emit({ accountPresentmentUrl: this.accountData!.accountPresentmentUrl! });
}
```

---

## HTTP

- All URLs are relative: `api/...`
- Define `basePath` constant on the service
- Encode dynamic path segments: `encodeURIComponent(id)`
- Strong typing on both request and response:

```typescript
export interface PeopleSearchRequest {
  keyword: string;
  limit?: number;
  skip?: number;
  sortBy?: string;
  sortDescending?: boolean;
}

return lastValueFrom(this.http.post<PeopleSearchResult>(this.basePath, request));
```

### Session interceptor

`session-expired.interceptor.ts` catches 403s on specific endpoints and redirects to `/session-end`. Returns `EMPTY` to suppress error propagation.

---

## Forms

Template-driven forms (`ngModel`) — no Reactive Forms:

```html
<input matInput [(ngModel)]="searchText" name="searchText" (keyup.enter)="onSearch()" />
```

Validation lives in the component:

```typescript
isShowResultsButtonDisabled = true;

onInputChanged(): void {
  const anyFilled = this.isAnyFieldFilled();
  const emailValid = this.isValidEmail(this.input.emailAddress);
  this.isShowResultsButtonDisabled = !anyFilled || !emailValid;
}
```

---

## Routing

### Lazy loading

```typescript
// app-routing.module.ts
{
  path: 'search',
  loadChildren: () => import('./search/search.module').then(m => m.SearchModule),
  canMatch: [nbsValidTenantMatchGuard, securityMatchGuard],
}
```

### Route guards

- `canMatch` — prevents route from appearing if tenant/access invalid (early)
- `canActivate` — guards component load (late, per child route)

### Route `data` metadata

```typescript
// people-search-routing.module.ts
data: {
  activeMenuItemId: 'people-search',         // highlights left nav item
  menuConfig: 'payment-platform',            // which nav menu to show
  securityCheck: (access: SecurityAccess) => !!access.canUsePeopleSearch,
  featureFlagCheck: () => flagService.isEnabled('people-search'),
}
```

### Route params

```typescript
this.route.paramMap.subscribe(params => {
  this.personId = params.get('personid') ?? '';
  void this.loadData();
});
```

### Navigation

```typescript
// With query params (merge keeps existing)
await this.router.navigate([], {
  relativeTo: this.route,
  queryParams: { selectedTab: 'user' },
  queryParamsHandling: 'merge',
});

// Absolute navigation
void this.router.navigate(['/person-dashboard', personId]);
```

---

## Shared Module

`SharedModule` centralises all Material + NBS imports/exports. Every feature module imports `SharedModule` instead of importing Material modules individually.

```typescript
// shared.module.ts
@NgModule({
  imports: [...ngModules, ...materialModules, ...xlr8Modules],
  exports: [...ngModules, ...materialModules, ...xlr8Modules],
  providers: [{ provide: INbsCountryAccessor, useClass: CountryAccessor }],
})
export class SharedModule {}
```

---

## NBS Library Usage

### Components

| Component | Usage |
|-----------|-------|
| `<npds-heading>` | `<npds-heading as="h1" size="xl">Title</npds-heading>` |
| `<npds-button>` | `<npds-button variant="primary" size="lg">Search</npds-button>` |
| `<npds-icon>` | `<npds-icon glyph="magnifying-glass"></npds-icon>` |
| `<npds-icon-button>` | Icon-only action buttons |

### Services

| Service | Purpose |
|---------|---------|
| `NbsTenantInfoService` | `isInternalTenant()`, current tenant context |
| `NbsGlobalNbsService` | `menuInfo`, `themeInfo`, `securityAccess` |
| `NbsOverlayService` | Open overlays/modals |
| `NbsSnackbarService` | Toast notifications |
| `NbsThemeService` | Apply theme from global service |

### App bootstrap pattern

```typescript
// app.component.ts
const globalNbsService = inject(NbsGlobalNbsService);
inject(NbsPendoService).initializePendo('Customer Service Portal');
inject(NbsThemeService).setTheme(globalNbsService.themeInfo);
NbsXlr8PageTemplateService.setHeaderMenu(globalNbsService.menuInfo['global-header-v2']?.items ?? []);
```

---

## Template Patterns

### Structural directives

```html
<div *ngIf="isLoading"><mat-progress-bar mode="indeterminate"></mat-progress-bar></div>
<app-person-card *ngFor="let person of displayedResults" [personSource]="person"></app-person-card>
```

### Material + NBS composition

```html
<mat-form-field appearance="outline">
  <npds-icon glyph="magnifying-glass" matPrefix></npds-icon>
  <input matInput [(ngModel)]="searchText" (keyup.enter)="onSearch()" />
  <mat-hint>{{ hintText }}</mat-hint>
</mat-form-field>
```

### Class and style binding

```html
[ngClass]="{'error-field': isBlankSearch}"
[class.error-hint]="isBlankSearch"
[disabled]="isLoading"
```

### Accessibility

```html
<npds-icon role="button" tabindex="0" aria-label="Clear search"></npds-icon>
<input i18n-placeholder="@@searchPlaceholderId" placeholder="Search" />
```

---

## i18n

- Source locale: `en` — additional: `es`, `eo`
- All user-visible strings go through `TranslationService` using `$localize`:

```typescript
// translation.service.ts
getPeopleSearchResultsMessage(count: number, term: string) {
  if (count === 1) {
    return $localize`:@@peopleSearch.singular:${count} result for "${term}"`;
  }
  return $localize`:@@peopleSearch.plural:${count} results for "${term}"`;
}
```

- Extract strings: `npm run i18n:extract`
- Build per locale: `npm run build-en | build-es | build-eo`
- Separate dist outputs: `dist-en/`, `dist-es/`, `dist-eo/`

---

## Key Config

| Setting | Value |
|---------|-------|
| `baseHref` | `/web/customer-service-portal/` |
| TypeScript | `strict: true`, `noUnusedLocals`, `noImplicitReturns` |
| Path aliases | `@app/*`, `@env/*`, `@testing/*` |
| Build budgets | 2MB initial warning / 5MB error; 6KB component style warning |

---

## Architectural Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Component type | NgModule (not standalone) | SharedModule re-export pattern |
| State management | Component class properties | Simpler than NgRx for this complexity level |
| Async | `lastValueFrom()` → Promise | Enables `async/await`; no manual unsubscribe |
| Change detection | `OnPush` on leaf components | Performance in result lists |
| Error handling | Bubble to caller; log at call site | Per-component flexibility |
| Forms | Template-driven (`ngModel`) | Sufficient for form complexity here |
| Route guards | `canMatch` + `canActivate` | `canMatch` hides route; `canActivate` guards load |
| Lazy loading | Per feature module | Reduces initial bundle |
| Logging | `TraceService` → Datadog | Production RUM + structured error context |
