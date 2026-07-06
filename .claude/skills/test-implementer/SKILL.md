---
name: test-implementer
description: Rules for writing complete, runnable NBS tests — unit, integration, SPA, and E2E
type: agent-skill
---

# Test Implementer

Write complete, runnable tests. No stubs. No placeholders. No `// TODO` comments.

## Before Writing Any Test

Read 1–2 existing test classes in the same test project. Lock in:
- Class declaration and constructor pattern (how dependencies are mocked and injected)
- `Mock<T>` declaration and setup style (`Setup`, `Returns`, `ReturnsAsync`)
- Arrange/Act/Assert structure and blank line separation
- Assertion style (`result.Should().Be(...)`, `.BeEquivalentTo(...)`, `.ThrowAsync<...>()`)
- Test method naming: `MethodName_Condition_ExpectedResult`
- Any shared fixtures, base classes, or helper methods

All new tests must be indistinguishable in style from existing ones.

## Coverage Rules

- 1 happy path per AC item
- 1–2 edge cases per AC item — only those listed in the plan
- Do not add extra tests beyond the plan

## Unit Tests (xUnit + Moq + FluentAssertions)

```csharp
[Fact]
public async Task MethodName_Condition_ExpectedResult()
{
    // Arrange
    var input = new FooRequest { /* real values, not null placeholders */ };
    _mockDependency.Setup(x => x.DoSomething(input)).ReturnsAsync(new Bar { Id = "abc-123" });

    // Act
    var result = await _sut.MethodName(input);

    // Assert
    result.Should().NotBeNull();
    result.Id.Should().Be("abc-123");
}
```

Rules:
- Real, meaningful values in Arrange — not `"test"`, `0`, or `null` unless the scenario tests null
- Only assert what the scenario is testing — no unrelated property assertions
- Edge cases: Arrange must set up the exact condition (null param, empty list, duplicate key, wrong tenant)
- Exception tests: `Func<Task> act = async () => await _sut.Method(input); await act.Should().ThrowAsync<ArgumentException>()`
- Time-dependent logic: freeze time with `Nbs.Framework.TimeTravel` before Act

## Integration Tests (xUnit + WebApplicationFactory)

- Use existing `WebServer` helper pattern from the test project — not raw `WebApplicationFactory`
- Use `appsettings.IntegrationTests.json` values — never hardcode connection strings
- Write full request construction: `HttpClient` call with method, path, headers, body
- Write full response assertions: status code AND body/property assertions
- Swap real services with `webServer.OverrideTestService<T>()` only for external services — DB and messaging stay real
- No `UseInMemoryDatabase` bypasses

## Angular Tests (Jasmine + Karma)

- Full `describe`/`it` blocks with complete `TestBed.configureTestingModule` setup
- Mirror existing specs: `HttpClientTestingModule`, `RouterTestingModule`, `@nbs/ng-testing` helpers if present
- Full `expect(...)` assertions — never `expect(true).toBe(true)`
- Handle async: `fakeAsync`/`tick`, or `async`/`await`, or `done` callback

## File Handling

- File already exists → add new methods inside the existing class, grouped by method under test
- File does not exist → create with full class scaffold (usings, namespace, class declaration, constructor) mirrored from an existing test class in the same project
- Never create a new file if an existing test class already covers the same class under test

## Updating Existing Tests

For each broken test from the plan:
1. Read the test file
2. Find the exact test method
3. Apply only the fix described (update mock setup, fix assertion, add parameter)
4. Do not change the test's intent
5. Do not delete tests — flag truly obsolete ones for human decision

## Using Directives Check

When removing a mock from a test constructor: check each removed `using` individually.
A namespace may cover multiple types — removing a using may break other code in the file.
Only remove a using if **no** type from that namespace remains in the file.
