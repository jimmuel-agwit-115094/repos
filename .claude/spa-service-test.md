# SPA / Frontend Tests (Unit)

Two distinct flavors: C# unit tests for SPA-adjacent backend code (localization, contracts), and Angular Karma unit tests for the Angular SPA itself.

---

## C# SPA Unit Tests (`WebSpa.UnitTests`)

Present in some repos (e.g., EventHub). Tests C# code that supports the SPA:
- Localization service resolution (`WebSpa.Localization`)
- Enum localization mappings
- `Client.Contracts` type correctness

**Tech:** xUnit + Moq + FluentAssertions — same stack as [backend-service-test.md](backend-service-test.md).

**Project refs:** `Client.Contracts`, `WebSpa.Localization` — no Angular involved.

```bash
dotnet test test/WebSpa.UnitTests
```

---

## Angular Karma Unit Tests

Angular component/service unit tests inside each SPA's workspace. Run via Angular CLI.

**Location:** Inside the service's Angular workspace:
- `src/WebSpa/` (most services) — Karma specs alongside components (`*.spec.ts`)
- `xlr8AngularToolkit/workspace/` — toolkit component specs
- `xlr8PageTemplate/workspace/` — template component specs

**Tech:**
- **Runner:** Karma (browser: Chrome / ChromeHeadless in CI)
- **Framework:** Jasmine (default Angular testing)
- **Shared helpers:** `@nbs/ng-testing` from `Framework.Web.AngularTesting`
- **Mocking:** Angular `TestBed`, `provideHttpClient(withInterceptorsFromDi()) + provideHttpClientTesting()` (prefer over deprecated `HttpClientTestingModule`), `RouterTestingModule`

**Run commands (from `workspace/` or `src/workspace/`):**

```bash
npm test                  # Chrome (interactive)
npm run test-headless     # ChromeHeadless (CI)
```

**CI pipeline:** Runs as part of `azure-pipelines-ci-npm.yaml` or the main CI pipeline.

---

## Shared Angular Testing Library (`Framework.Web.AngularTesting`)

Published as `@nbs/ng-testing` npm package. Provides:
- Shared Karma config utilities
- Playwright v1 and v2 TypeScript configs (consumed by E2E tests)
- Shared test utility functions

Consumed by all SPA test projects and `WebSpa.E2ETests` packages (via `@nbs/ng-testing` devDependency).

---

## Per-Repo Notes

| Repo | SPA Test Type |
|------|--------------|
| EventHub | `WebSpa.UnitTests` (C#) + Angular Karma specs in `WebSpa` |
| Banking | Angular Karma specs in `WebSpa` |
| Checkout | Angular Karma specs in `WebSpa` |
| Payments | Angular Karma specs in `WebSpa` + Blazor (manual) |
| Remittance | Angular Karma specs in `WebSpa` |
| xlr8AngularToolkit | `npm test` in `workspace/` |
| xlr8PageTemplate | `npm test` in `workspace/` |
