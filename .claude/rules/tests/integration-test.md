# Integration Tests

Tests that spin up the real ASP.NET Core host (in-memory) against real MongoDB, with real DI wiring. Covers WebApi endpoints. Job integration tests follow the same pattern.

## Project Naming

| Pattern | Example |
|---------|---------|
| `WebApi.IntegrationTests` | `Banking/test/WebApi.IntegrationTests` |
| `{Job}.IntegrationTests` | `Banking/test/CaptureJob.IntegrationTests` |
| `{Service}.Web.IntegrationTests` | `Passport/test/Passport.Web.IntegrationTests` |

## Tech Stack

- **Test framework:** xUnit
- **Assertions:** FluentAssertions + custom assertion extensions (`AssertionExtensions.cs`)
- **Host:** `Microsoft.AspNetCore.Mvc.Testing` → `WebApplicationFactory<TStartup>`
- **Messaging (in-memory):** `Nbs.Framework.Messaging.InMemory` — replaces real Azure Event Hubs
- **Messaging test helpers:** `Nbs.Framework.Messaging.Testing`, `Nbs.Framework.Messaging.TimeTravel`
- **Outbox:** `Nbs.Framework.Messaging.Outbox.MongoDb`
- **Coverage:** coverlet.collector

## Project Structure

```
WebApi.IntegrationTests/
  appsettings.IntegrationTests.json   # Test config (DB, auth, service URLs)
  TestFixture.cs                      # IAsyncLifetime — host setup, DI, seeding orchestration
  TestCollectionFixture.cs            # xUnit ICollectionFixture<TestFixture> definition
  BaseTest.cs                         # Abstract base: typed client properties, auth helpers
  WebServer.cs                        # WebApplicationFactory<TStartup> wrapper
  MockUserContext.cs                  # IUserContext mock (default userId = 1)
  IntegrationTestSettings.cs          # Config POCO
  AssertionExtensions.cs              # Custom FluentAssertions helpers
  DataSeed/                           # IAsyncLifetime seeders per entity type
    Seed{EntityType}Data.cs
  DependentServiceIds/                # Lazy service ID fetchers (@RegisterService)
  LogData.cs                          # Side-effect log accessor + cleanup
  {Feature}Tests/                     # Concrete test classes grouped by feature
    {Feature}ClientTests.cs
```

Reference: `CustomerServicePortal/test/WebApi.IntegrationTests/`

---

## Fixture & Collection Pattern

### TestFixture (`IAsyncLifetime`)

One fixture per test collection — created once, shared across all tests.

**`InitializeAsync` responsibilities:**
1. Load `appsettings.IntegrationTests.json`; resolve `{{Placeholder}}` config values
2. Build `WebServer<Startup>` when `UseInMemoryWebServer=true`
3. Register real dependencies (MongoDB, typed HTTP clients, accessors, services, seeders)
4. Call `InitializeAsync()` on all `IAsyncLifetime` seeders (inserts test data)
5. Create authenticated HTTP clients (bearer via `AddUserIdForwarding`)

**`DisposeAsync` responsibilities:**
- Dispose all seeders in reverse (each seeder deletes its own data)
- Tear down `WebServer` and HTTP handler
- Flush Serilog

### Collection Fixture

```csharp
// TestCollectionFixture.cs
[CollectionDefinition(TestFixture.CollectionName)]  // "CustomerServicePortalCollection"
public class TestCollectionFixture : ICollectionFixture<TestFixture> { }
```

---

## Base Test Class

```csharp
[Collection(TestFixture.CollectionName)]
public class UserSearchClientTests : BaseTest
{
    private readonly SeedUserSearchData _seedData;

    public UserSearchClientTests(TestFixture fixture) : base(fixture)
    {
        _seedData = fixture.SeedUserSearchData;  // seeder populated in InitializeAsync
    }

    [Fact]
    public async Task Quick_search_returns_seeded_result()
    {
        var result = await UserSearchClient.SearchAsync(request, tenantId, default);
        result.Users.Should().HaveCountGreaterThanOrEqualTo(1);
    }
}
```

`BaseTest` exposes:
- `TestFixture` — shared fixture reference
- `HttpClient` — raw authenticated HTTP client
- Typed client properties (authenticated): `UserSearchClient`, `PersonSearchClient`, etc.
- Unauthenticated variants for 401/403 tests: `UnauthenticatedUserSearchClient`, etc.
- `InMemoryWebServer` — `bool` to skip remote-only tests
- `TenantId` — lazy awaitable tenant ID fetcher

---

## WebServer Helper

Present in all repos. Wraps `WebApplicationFactory<TStartup>`:

```csharp
// Override config at startup
webServer.OverrideConfigurationSetting("SomeSetting:Key", "value");

// Swap a real service for a test double
webServer.OverrideTestService<IMyService>(myMock);
```

Config loaded from `appsettings.IntegrationTests.json`. Key settings:

```json
{
  "UseInMemoryWebServer": true,
  "TenantKey": "lincolnschool",
  "MongoDbAccessorSettings": {
    "ConnectionString": "mongodb://localhost:27017/customerserviceportal"
  },
  "ServiceConnections": {
    "Default": {
      "AuthorityUrl": "https://login-alpha1.nelnet.net/",
      "ClientId": "...",
      "ClientSecret": "...",
      "Scope": "Passport.Secured.API"
    },
    "TenantClient": { "ServiceUrl": "https://alpha.factsmgt.com/api/tenant" }
  }
}
```

Config placeholders (`{{AuthSettings:AuthorityUrl}}`) are resolved by `ReplaceConfigurationPlaceholders()` at startup — not standard .NET config syntax.

---

## Data Seeding

Seeders inherit from the production accessor base class and implement `IAsyncLifetime`:

```csharp
// DataSeed/SeedUserSearchData.cs
public record SeedUserSearchData : UserSearchAccessor, IAsyncLifetime
{
    public async Task InitializeAsync()
    {
        await Collection.InsertManyAsync(UserList);  // direct MongoDB insert
    }

    public async Task DisposeAsync()
    {
        var ids = UserList.Select(x => x.UserAccountId);
        await Collection.DeleteManyAsync(FilterBuilder.In(x => x.UserAccountId, ids));
    }

    public IEnumerable<UserSearchResultDoc> UserList => new[]
    {
        new UserSearchResultDoc
        {
            UserAccountId = "1523648",
            FirstName = "BlackTest",
            LastName = "Maelstrom",
            TenantIds = new List<Guid> { Guid.Parse("5b241101-...") },
            IsActive = true,
            UserTypes = new[] { UserType.Consumer }
        }
    };
}
```

**Rules:**
- Each seeder inserts in `InitializeAsync`, deletes in `DisposeAsync` — DB is never dropped
- Seeders registered as singletons in `TestFixture`; accessed via fixture properties
- **Do not run seeders in parallel** — parallel inserts on a new DB cause MongoDB `WriteConflict` on outbox collection creation
- Log data (`LogData.cs`) is cleaned up manually inside test assertions, not in the seeder

---

## Authentication

- Bearer token injected via `AddUserIdForwarding()` — no manual JWT generation
- `MockUserContext` provides default `UserId = 1`:

```csharp
public record MockUserContext : IUserContext
{
    public long? MockUserId { get; set; } = 1;
    public long UserId => MockUserId ?? throw new NotImplementedException();
}
```

- For 401/403 tests, use the unauthenticated client variant from `BaseTest`:

```csharp
var result = await UnauthenticatedUserSearchClient.SearchAsync(request, tenantId, default);
result.Should().Throw401Unauthorized();  // custom assertion extension
```

- Skip tests that only apply to remote (non-in-memory) mode:

```csharp
if (InMemoryWebServer) { await HealthCheckTest("health/ready"); }
```

---

## Test Patterns

### Theory with MemberData

```csharp
[Theory(DisplayName = "Quick Search Should return result")]
[MemberData(nameof(GetUserSearchRequests))]
public async Task Quick_search_shouldreturn_expected_result(UserSearchRequest request)
{
    var result = await UserSearchClient.SearchAsync(request, mockTenantId, default);
    result.Users.Should().HaveCountGreaterThanOrEqualTo(1);
}

public static IEnumerable<object?[]> GetUserSearchRequests()
{
    yield return new[] { new UserSearchRequest { Keyword = "blacktest", Limit = 10 } };
    yield return new[] { new UserSearchRequest { Keyword = "strom", Limit = 5 } };
}
```

### Log Side-Effect Verification

```csharp
private readonly LogData _logData;

// In test:
var logs = await _logData.GetAll();
logs.Should().HaveCountGreaterThanOrEqualTo(1);
await _logData.DisposeAsync();  // manual cleanup — not handled by seeder DisposeAsync
```

### Custom Assertion Extensions

```csharp
response.Should().Throw401Unauthorized();
response.Should().Throw400BadRequest();
response.Should().Throw404NotFound();
```

Defined in `AssertionExtensions.cs` — check this file before writing raw status code assertions.

---

## Dependency Services (Local)

Integration tests require live MongoDB. When running against real external services (non-in-memory mode):

```powershell
Invoke-Xlr8GoToDirectory Banking
Invoke-Xlr8ServiceRunner rm *
Invoke-Xlr8ServiceRunner add deps    # integration tests (no this service)
Invoke-Xlr8ServiceRunner add . deps  # full stack including this service
```

See [dependencies.md](../microservice-setup/dependencies.md) for each service's required deps.

## Run Commands

```bash
dotnet test test/WebApi.IntegrationTests
dotnet test test/CaptureJob.IntegrationTests
dotnet test {Repo}.sln --filter "Category=Integration"
```

---

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
| CustomerServicePortal | `WebApi.IntegrationTests` — 6 seeder classes, log side-effect verification, no job tests |
