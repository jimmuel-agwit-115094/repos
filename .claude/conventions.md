# Conventions

Shared patterns across all NBS repos. Per-service deviations documented in that service's own `.claude/` files.

---

## C# / .NET

- **Framework:** .NET (version managed per repo via `global.json`; CustomerServicePortal is .NET 10)
- **Package versions:** Centralized in `Directory.Packages.props` — do not add `Version=` to individual `.csproj` references
- **Build props:** `Directory.Build.props` at repo root applies to all projects
- **NuGet:** Private feed config in `nuget.config`
- **Naming:**
  - Projects: `{Repo}.{Layer}` (e.g., `Banking.Services`, `Banking.Accessors`)
  - Contracts DTOs: PascalCase, suffix `Request` / `Response` / `Event`
  - Events: past-tense (e.g., `PaymentProcessed`, `OrderCreated`)
- **Layers (strict dependency direction):**
  ```
  WebApi → Services → Accessors → (external)
  WebApi → Contracts (DTOs only)
  Client.Contracts → Contracts (shared interfaces)
  ```
- **Auth:** `[Authorize]` + `[SecuredFunction]` attributes from Passport client

---

## Angular / TypeScript

- **Angular CLI workspace** pattern — all Angular projects under `workspace/` or `src/workspace/`
- **ESLint** config via `eslint.config.mjs` (flat config format)
- **i18n:** Three locales — `en`, `es`, `eo` (Esperanto for testing). Extract with `npm run i18n:extract`
- **Base URL pattern:** `/web/{service-name}/`
- **Build:** Gulp pipeline wraps `ng build` for library builds; `npm run build` at workspace root
- **Naming:**
  - Components: `kebab-case` selectors, `PascalCase` class names
  - Services: `{Feature}Service`
  - State/store: follow existing pattern per service

---

## CI / CD

- **Azure DevOps Pipelines** — multiple yaml files per repo:
  - `azure-pipelines-pr.yml` — PR validation (build + tests)
  - `azure-pipelines-ci.yml` — CI on merge (build + push artifacts)
  - `azure-pipelines-ci-packages.yaml` — publish NuGet/npm packages
  - `azure-pipelines-update-packages.yaml` — automated dependency updates
- **Versioning:** `version.json` at repo root (Nerdbank.GitVersioning pattern)
- **Helm charts:** `charts/` folder per repo for Kubernetes deployment
- **Service Now:** Each repo needs its own CMDB Configuration Item ID in pipelines

---

## Database

- **MongoDB Atlas** — primary datastore for most services
- **Migrations:** `DbScripts/` folder with numbered seed/migration scripts
- **DbUpdater:** Some services (Remittance) have a `DbUpdater` console app for running migrations

---

## Local Development

- **Xlr8 Tools** (PowerShell): `Invoke-Xlr8GoToDirectory`, `Invoke-Xlr8ServiceRunner` — see [commands.md](commands.md)
- **Service runner config:** `Invoke-Xlr8ServiceRunner add deps` — wires up dependency services
