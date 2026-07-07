# SPA / Frontend Tests (Unit)

Two distinct flavors: C# unit tests for the SPA's backend code (controllers, services, authorization), and Angular Karma/Jasmine specs for the Angular app itself.

---

## C# SPA Unit Tests (`WebSpa.UnitTests`)

Tests C# code in the `WebSpa` project — controllers, services, and authorization attributes. **No Angular involved.**

Reference: `CustomerServicePortal/test/WebSpa.UnitTests/`

### What Gets Tested

| Folder | Tests |
|--------|-------|
| `Controllers/` | HTTP endpoint logic — return types, status codes, data transforms |
| `Services/` | Business logic helpers — tenant resolution, security filtering, person detail building |
| `Authorization/` | `[TenantSecuredFunction]` attribute presence and permissions via reflection |

**Not tested here:** Angular components, database layer, message handlers.

### Tech Stack

- **Framework:** xUnit (`[Fact]`, `[Theory]`, `[InlineData]`)
- **Assertions:** FluentAssertions
- **Mocking:** Moq
- **Coverage:** coverlet.collector
- **Logging:** `NullLogger<T>.Instance` for dependencies that log

### Project Structure

```
WebSpa.UnitTests/
  Authorization/
    AuthorizationTestHelpers.cs          # Reflection helper: extract [TenantSecuredFunction] permissions
    {Feature}ControllerAuthorizationTests.cs
  Controllers/
    {Feature}ControllerTests.cs
  Services/
    {ServiceName}Tests.cs
```

### Test Class Pattern

No base class — each test class is self-contained. Constructor initializes all mocks and the system under test.

```csharp
public class PersonControllerTests
{
    // Controllers: MockBehavior.Strict — fail if called unexpectedly
    private readonly Mock<IPersonClient> _personClientMock;
    private readonly Mock<ITenantHelper> _tenantHelperMock;
    private readonly PersonController _controller;

    public PersonControllerTests()
    {
        _personClientMock   = new Mock<IPersonClient>(MockBehavior.Strict);
        _tenantHelperMock   = new Mock<ITenantHelper>(MockBehavior.Strict);

        _controller = new PersonController(
            _personClientMock.Object,
            _tenantHelperMock.Object);

        // Required when controller reads Request.Host / Request.Scheme / Request.PathBase
        _controller.ControllerContext = new()
        {
            HttpContext = new DefaultHttpContext()
        };
    }

    // Organize with #region per method under test
    #region GetPersonDetail Tests

    [Fact]
    public async Task GetPersonDetail_WithValidPersonId_ReturnsOk()
    {
        // Arrange
        var personId = "person123";
        var tenantId = Guid.NewGuid();

        _tenantHelperMock
            .Setup(x => x.GetRootTenantIdAsync(CancellationToken.None))
            .ReturnsAsync(tenantId);
        _personClientMock
            .Setup(x => x.GetPersonAsync(tenantId, personId, CancellationToken.None))
            .ReturnsAsync(new Person { PersonId = personId });

        // Act
        var result = await _controller.GetPersonDetail(personId, CancellationToken.None);

        // Assert
        var ok = result.Result.Should().BeOfType<OkObjectResult>().Subject;
        ok.Value.Should().BeOfType<PersonDetail>();

        _tenantHelperMock.Verify(x => x.GetRootTenantIdAsync(CancellationToken.None), Times.Once);
        _personClientMock.Verify(x => x.GetPersonAsync(tenantId, personId, CancellationToken.None), Times.Once);
    }

    [Fact]
    public async Task GetPersonDetail_WhenTenantIdIsNull_ReturnsBadRequest()
    {
        _tenantHelperMock
            .Setup(x => x.GetRootTenantIdAsync(CancellationToken.None))
            .ReturnsAsync((Guid?)null);

        var result = await _controller.GetPersonDetail("p1", CancellationToken.None);

        var bad = result.Result.Should().BeOfType<BadRequestObjectResult>().Subject;
        bad.Value.Should().Be("Tenant ID is required.");

        // Verify downstream mocks were NOT called
        _personClientMock.Verify(
            x => x.GetPersonAsync(It.IsAny<Guid>(), It.IsAny<string>(), It.IsAny<CancellationToken>()),
            Times.Never);
    }

    #endregion
}
```

**Mock strategy:**
- **Controllers → `MockBehavior.Strict`** — every unexpected call fails the test
- **Services → `MockBehavior.Loose`** (default) — unconfigured calls return defaults

### Service Test Pattern

```csharp
public class PersonDetailHelperTests
{
    private readonly Mock<IPersonService> _personServiceMock;   // Loose
    private readonly PersonDetailHelper _sut;

    public PersonDetailHelperTests()
    {
        _personServiceMock = new Mock<IPersonService>();
        _sut = new PersonDetailHelper(
            _personServiceMock.Object,
            NullLogger<PersonDetailHelper>.Instance);  // never new Mock<ILogger>
    }

    // Extract common setup to private helper methods, not to a base class
    private void SetupPersonServiceDefaults()
    {
        _personServiceMock
            .Setup(x => x.GetDisplayName(It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()))
            .Returns(string.Empty);
        _personServiceMock
            .Setup(x => x.GetAddress(It.IsAny<IEnumerable<PostalAddress>>()))
            .Returns(string.Empty);
    }
}
```

### Authorization Test Pattern

Uses reflection — tests that the attribute *exists* and carries correct permission constants. Runtime enforcement is covered by integration tests.

```csharp
public class TenantSearchControllerAuthorizationTests
{
    [Theory]
    [InlineData(nameof(TenantSearchController.GetTenantLastSearch))]
    [InlineData(nameof(TenantSearchController.SaveLastSearch))]
    public void AllMethods_HaveCorrectAuthorizationAttribute(string methodName)
    {
        var method = typeof(TenantSearchController).GetMethod(methodName);
        var attribute = method?.GetCustomAttributes(typeof(TenantSecuredFunctionAttribute), false)
            .FirstOrDefault() as TenantSecuredFunctionAttribute;

        attribute.Should().NotBeNull($"{methodName} should be protected");

        // AuthorizationTestHelpers.cs: custom helper in Authorization/ folder
        var permissions = AuthorizationTestHelpers.GetTenantSecuredFunctionPermissions(method);

        permissions.Should().Contain(SecuredFunctions.MaintainExternalTenants);
        permissions.Should().Contain(SecuredFunctions.ReadTenants);
    }
}
```

**`AuthorizationTestHelpers.cs`** — shared across all authorization test classes. Always check this file before writing permission assertions.

### Naming Convention

`{MethodName}_{Condition}_{ExpectedResult}` — e.g.:
- `GetPersonDetail_WithValidPersonId_ReturnsOk`
- `GetPersonDetail_WhenTenantIdIsNull_ReturnsBadRequest`
- `BuildPersonDetailAsync_HierarchyMatch_SelectsMatchedTenant`

### Run Command

```bash
dotnet test test/WebSpa.UnitTests
```

---

## Angular Karma Unit Tests

Angular component/service specs inside the SPA workspace. Colocated with components.

**Location:**
- CSP: `src/WebSpa/ClientApp/src/app/**/*.spec.ts`
- Other services: `src/WebSpa/*.spec.ts` or `src/workspace/*.spec.ts`
- Libraries: `xlr8AngularToolkit/workspace/`, `xlr8PageTemplate/workspace/`

### Tech Stack

- **Runner:** Karma (Chrome interactive / ChromeHeadless in CI)
- **Framework:** Jasmine
- **Shared helpers:** `@nbs/ng-testing` from `Framework.Web.AngularTesting`
- **HTTP mocking:** `provideHttpClient(withInterceptorsFromDi())` + `provideHttpClientTesting()` — **not** deprecated `HttpClientTestingModule`

### TestBed Pattern

```typescript
describe('PeopleSearchService', () => {
    let service: PeopleSearchService;
    let httpMock: HttpTestingController;

    beforeEach(async () => {
        await TestBed.configureTestingModule({
            providers: [
                PeopleSearchService,
                provideHttpClient(withInterceptorsFromDi()),
                provideHttpClientTesting(),
                { provide: NbsGlobalNbsService, useValue: { menuInfo: {}, themeInfo: 'light' } }
            ]
        }).compileComponents();

        service  = TestBed.inject(PeopleSearchService);
        httpMock = TestBed.inject(HttpTestingController);
    });

    afterEach(() => {
        httpMock.verify();  // ensures all expected requests were matched
    });

    it('should POST to correct endpoint', async () => {
        const request: PeopleSearchRequest = { keyword: 'John', limit: 10 };
        const mockResponse: PeopleSearchResult = { resultCount: 1, people: [] };

        const promise = service.search(request);

        const req = httpMock.expectOne('api/people-search');
        expect(req.request.method).toBe('POST');
        expect(req.request.body).toEqual(request);
        req.flush(mockResponse);

        const result = await promise;
        expect(result).toEqual(mockResponse);
    });
});
```

**Key rules:**
- `httpMock.verify()` in every `afterEach` that uses HTTP
- Use `async/await` — not `fakeAsync`/`tick`
- All NBS service providers (`NbsGlobalNbsService`, etc.) stubbed with `useValue`
- `provideRouter([])` for components that inject `Router`

### Run Commands

```bash
# From workspace/ or src/workspace/
npm test              # Chrome (interactive)
npm run test-headless # ChromeHeadless (CI)
```

**CI:** Runs as part of `azure-pipelines-ci-npm.yaml` or the main CI pipeline.

---

## Shared Angular Testing Library (`Framework.Web.AngularTesting`)

Published as `@nbs/ng-testing`. Provides:
- Shared Karma config utilities
- Playwright v1 and v2 TypeScript configs (consumed by E2E projects)
- Shared test utility functions

All SPA test projects and `WebSpa.E2ETests` packages consume this via `@nbs/ng-testing` devDependency.

---

## Per-Repo Notes

| Repo | C# `WebSpa.UnitTests` | Angular Karma specs |
|------|-----------------------|---------------------|
| CustomerServicePortal | Controllers, Services, Authorization (reflection-based) | `src/WebSpa/ClientApp/src/app/**/*.spec.ts` |
| EventHub | Localization, enum mappings | `src/WebSpa/*.spec.ts` |
| Banking | — | `src/WebSpa/*.spec.ts` |
| Checkout | — | `src/WebSpa/*.spec.ts` |
| Payments | — | `src/WebSpa/*.spec.ts` + Blazor (manual) |
| Remittance | — | `src/WebSpa/*.spec.ts` |
| xlr8AngularToolkit | — | `workspace/` — `npm test` |
| xlr8PageTemplate | — | `workspace/` — `npm test` |
