# Backend Service Tests (Unit)

Unit tests for Services, Accessors, Client, Jobs, and WebApi layers. No real DB, HTTP, or message bus — dependencies mocked.

## Project Naming

| Pattern | Example |
|---------|---------|
| `{Layer}.UnitTests` | `Banking/test/Services.UnitTests` |
| `{Service}.{Layer}.UnitTests` | `Passport/test/Passport.Services.UnitTests` |
| `{Job}.UnitTests` | `ScheduledPayments/test/RecordJob.UnitTests` |

## Tech Stack

- **Test framework:** xUnit
- **Assertions:** FluentAssertions
- **Mocking:** Moq
- **Coverage:** coverlet.collector
- **Time:** `Nbs.Framework.TimeTravel` — freeze/advance time in tests
- **DI:** `Microsoft.Extensions.DependencyInjection` wired manually per test class

## What Gets Tested Here

- Service layer business logic (pure logic, mocked accessors/clients)
- Accessor layer data mapping / query construction (mocked DB driver)
- Client layer response parsing
- Job logic (mocked service dependencies)
- Localization resource resolution (`WebApi.Localization`, `WebSpa.Localization`)

## Project References

Each unit test project references only the layer under test:

```xml
<ProjectReference Include="..\..\src\Services\Services.csproj" />
```

Never references `WebApi` or `BackgroundServices` directly unless testing their units in isolation.

## Run Command

```bash
dotnet test test/{Layer}.UnitTests
# or entire solution
dotnet test {Repo}.sln --filter "Category=Unit"
```

## Per-Repo Notes

| Repo | Unit Test Projects |
|------|-------------------|
| Banking | `Services.UnitTests`, `Accessors.UnitTests`, `WebApi.UnitTests` |
| Checkout | `Services.UnitTests`, `Accessors.UnitTests` |
| Payments | `Services.UnitTests`, `Accessors.UnitTests`, `Client.UnitTests`, `SharedConstants.UnitTests` |
| Remittance | `Services.UnitTests` |
| ScheduledPayments | `Services.UnitTests`, `RecordJob.UnitTests` |
| Encryption | `Services.UnitTests` |
| EventHub | `Services.UnitTests`, `Accessors.UnitTests` |
| Passport | `Passport.Services.UnitTests`, `Passport.Accessors.UnitTests`, `Passport.Web.UnitTests`, `Passport.Client.AspNetCore.UnitTests`, `Passport.MemoryAuthorizationCache.UnitTests`, `Passport.RedisAuthorizationCache.UnitTests` |
| Tenant | `Services.UnitTests` |
| Menu | `Services.UnitTests` |
| ChangeHistory | `Services.UnitTests` |
