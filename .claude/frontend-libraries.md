# Frontend Libraries

Angular npm libraries shared across all NBS service SPAs. Built with Gulp + Angular CLI workspace pattern.

---

## xlr8AngularToolkit (`xlr8AngularToolkit/`)

**Purpose:** Main NBS Angular component/UI library. Published as npm packages consumed by all service SPAs.

**Workspace:** `workspace/`

**Published packages:**
- `ng-xlr8-toolkit` — component library
- `ng-xlr8-toolkit-styles` — SCSS styles/theme
- `demo` — live demo app (not published)

**Scripts:** `gulp` build pipeline; `npm run serve-demo` to develop locally on port `5028`.

**CI:** `azure-pipeline.yaml` (build + publish) + `azure-pr-pipeline.yaml`

---

## xlr8PageTemplate (`xlr8PageTemplate/`)

**Purpose:** Standard NBS page layout/shell — header, footer, sidebar. Wraps all SPA pages.

**Workspace:** `workspace/`

**Published packages:**
- `xlr8-page-template` — Angular component library
- `xlr8-page-template-npm` — npm-publishable variant
- `demo` — demo app

**CI:** `azure-pipelines.yml` (build) + `azure-pipelines-update-pendo.yaml` (Pendo analytics token update)

---

## Framework.Web.AngularTesting (`Framework.Web.AngularTesting/`)

**Purpose:** Shared Angular test utilities — Karma unit test helpers and Playwright E2E helpers. Imported by service SPA test projects.

**Package root:** `ng-testing/`

**Test configs:**
- `tsconfig-karma.json` — unit tests
- `tsconfig-playwright.json` / `tsconfig-playwright-v2.json` — E2E tests
- `tsconfig-shared-utilities.json` — shared utils

**CI:** `azure-pipelines.yml`

---

## PaymentMethodSelector (`PaymentMethodSelector/`)

**Purpose:** Embeddable payment method selection widget. Ships as both a .NET BFF and an Angular web component.

**Stack:** Hybrid — .NET `AspNetCore.Bff` + Angular `WebComponent` + Angular `WebSpa`.

**Note:** Has separate CI pipelines for npm artifacts and NuGet packages.
