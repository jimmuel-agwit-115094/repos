# Integration Tests

Tests that spin up the real ASP.NET Core host (in-memory) against real or in-memory MongoDB, with real DI wiring. Covers WebApi endpoints and background job processing.

## Project Naming

| Pattern | Example |
|---------|---------|
| `WebApi.IntegrationTests` | `Banking/test/WebApi.IntegrationTests` |
| `{Job}.IntegrationTests` | `Banking/test/CaptureJob.IntegrationTests` |
| `{Service}.Web.IntegrationTests` | `Passport/test/Passport.Web.IntegrationTests` |

## Tech Stack

- **Test framework:** xUnit + `Xunit.SkippableFact` (skip tests when deps unavailable)
- **Assertions:** FluentAssertions
- **Host:** `Microsoft.AspNetCore.Mvc.Testing` → `WebApplicationFactory<TStartup>`
- **Messaging (in-memory):** `Nbs.Framework.Messaging.InMemory` — replaces real Azure Event Hubs
- **Messaging test helpers:** `Nbs.Framework.Messaging.Testing`, `Nbs.Framework.Messaging.TimeTravel`
- **Outbox:** `Nbs.Framework.Messaging.Outbox.MongoDb`
- **Coverage:** coverlet.collector

## WebServer Helper (`Testing.Common/Setup/WebServer.cs`)

Present in Banking, Payments, and other repos. Wraps `WebApplicationFactory<T>`:

```csharp
// Override config per test
webServer.OverrideConfigurationSetting("SomeSetting:Key", "value");

// Swap a real service for a test double
webServer.OverrideTestService<IMyService>(myMock);
```

Custom config loaded from `appsettings.IntegrationTests.json` in the test project. Key settings:

```json
{
  "UseInMemoryWebServer": true,
  "TenantKey": "phubconfig1",
  "MessagingPrefix": "{{MachineName}}-"
}
```

## Dependency Services (Local)

Integration tests running against real external services (not in-memory mode) need dependent services running. Use Xlr8 Tools:

```powershell
Invoke-Xlr8GoToDirectory Banking
Invoke-Xlr8ServiceRunner rm *
Invoke-Xlr8ServiceRunner add deps    # in-memory integration tests
Invoke-Xlr8ServiceRunner add . deps  # full E2E mode
```

See [dependencies.md](dependencies.md) for each service's required deps.

## Run Command

```bash
dotnet test test/WebApi.IntegrationTests
dotnet test test/CaptureJob.IntegrationTests
# or entire solution
dotnet test {Repo}.sln --filter "Category=Integration"
```

## Per-Repo Notes

| Repo | Integration Test Projects |
|------|--------------------------|
| Banking | `WebApi.IntegrationTests`, `CaptureJob.IntegrationTests`, `TransmissionJob.IntegrationTests`, `NeedsVerificationJob.IntegrationTests`, `JobCoordinator.IntegrationTests` |
| Checkout | `WebApi.IntegrationTests`, `ExpireOrdersJob.IntegrationTests` |
| Payments | `WebApi.IntegrationTests`, `NeedsVerificationJob.IntegrationTests` |
| Remittance | `WebApi.IntegrationTests`, `RemittanceJob.IntegrationTests` |
| ScheduledPayments | `WebApi.IntegrationTests` |
| Encryption | `WebApi.IntegrationTests` |
| EventHub | `WebApi.IntegrationTests` |
| Passport | `Passport.Web.IntegrationTests`, `Passport.Web.Legacy.IntegrationTests`, `Passport.Web.Legacy.Sync.IntegrationTests`, `Passport.Web.DeprecatedApis.IntegrationTests`, `Passport.Job.IntegrationTests` |
| Tenant | `WebApi.IntegrationTests` |
| Menu | `WebApi.IntegrationTests` |
| ChangeHistory | `WebApi.IntegrationTests` |
