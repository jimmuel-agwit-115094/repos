# Commands

## Xlr8 Tools (PowerShell — Local Dev)

NBS PowerShell library. See [NBS PowerShell Libraries](https://dev.azure.com/nbsdev/Services/_git/PowerShell).

```powershell
# Navigate to a service directory
Invoke-Xlr8GoToDirectory <ServiceName>

# Manage local service runner (dependency services)
Invoke-Xlr8ServiceRunner rm *            # Remove all running services
Invoke-Xlr8ServiceRunner add deps        # Add dependency services (in-memory integration tests)
Invoke-Xlr8ServiceRunner add . deps      # Add this service + dependencies (E2E tests)
```

---

## .NET (per repo root)

```bash
dotnet build <Repo>.sln
dotnet test <Repo>.sln
dotnet restore --interactive             # For private NuGet feeds
```

---

## Angular / npm (from `workspace/` or `src/workspace/`)

```bash
npm install
npm run build          # Production build (all publishable libs)
npm run build-all      # Build all projects including demo
npm run serve-demo     # Dev server on port 5028
npm run test           # Karma unit tests (Chrome)
npm run test-headless  # Karma unit tests (headless CI)
npm run lint           # ESLint
npm run i18n:extract   # Extract i18n strings → XLF files
```

---

## Azure DevOps CI Pipelines (reference)

| Pipeline file | Purpose |
|---------------|---------|
| `azure-pipelines-pr.yml/yaml` | PR validation |
| `azure-pipelines-ci.yml/yaml` | CI on merge to main |
| `azure-pipelines-ci-packages.yaml` | Publish NuGet / npm packages |
| `azure-pipelines-ci-npm.yaml` | Publish npm-only packages |
| `azure-pipelines-update-packages.yaml` | Automated dependency bumps |
| `azure-pipelines-*-load-test.*` | Load/performance tests |

---

## Helm / Kubernetes (reference)

Charts in `charts/` per repo. Deployed via CI pipelines — do not deploy manually unless instructed.
