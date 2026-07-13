---
name: implementation-plan
description: Template for NBS implementation plan files. Populate every section and write to .claude/implementation-plan/{story-id}.md
type: agent-skill
---

Use this template to write the implementation plan. Populate every section. Omit a section only if it genuinely does not apply (e.g. no DB changes, no frontend). Never leave placeholder text in the output file.

Write to: `C:\neldevsrc\repos\.claude\implementation-plan\{story-id}.md`

---

```markdown
# {Story ID} — {Story Title}

## Story Context

**Work Item ID:** {id}
**Type:** {Bug / User Story / Task}
**Title:** {exact verbatim title}

### Description

{Full description from ADO — do not summarize. Paste verbatim.}

### Acceptance Criteria (verbatim)

1. {AC item 1 — exact wording from ADO}
2. {AC item 2 — exact wording from ADO}

### Requirements and Constraints

{Paste verbatim from story. Do not summarize.}

---

## Complexity Assessment

**Backend:** None / Low / Medium / High
**Frontend:** None / Low / Medium / High
**Database:** None / Low / Medium / High
**Testing:** Low / Medium / High
**Risk:** Low / Medium / High
**Estimated Complexity:** Low / Medium / High

---

## Affected Service(s)

| Service | Why affected |
|---------|-------------|
| {ServiceName} | {specific reason} |

---

## Pattern References

Files the developer should study before coding. These are the closest existing implementations to copy shape from.

| What | File | Reuse strategy |
|------|------|---------------|
| Controller pattern | `{path}` | Extend / Copy shape |
| Service pattern | `{path}` | Extend / Copy shape |
| Accessor pattern | `{path}` | Extend / Copy shape |
| Doc/model pattern | `{path}` | Extend / Copy shape |
| Angular component pattern | `{path}` | Extend / Copy shape |
| Angular service pattern | `{path}` | Extend / Copy shape |
| Unit test pattern | `{path}` | Copy shape |
| Integration test pattern | `{path}` | Copy shape |
| Angular spec pattern | `{path}` | Copy shape |

---

## Assumptions

List every implicit design decision made while producing this plan.

- {e.g. Existing endpoint will be extended rather than creating a new one}
- {e.g. Existing DTO covers the required fields — no new type needed}
- {e.g. Current authentication flow is unchanged}
- {e.g. No migration script needed — new field is nullable and defaults safely}

---

## Contracts and DTOs

Define every new or changed DTO, request, response, and event. Include all properties with types. If an existing type is being extended, show only the added/changed members and note the source type.

### {DtoName} (`{namespace or file path}`)

> **Action:** New type / Extending `{ExistingType}` / No change — reuse `{ExistingType}` as-is

```csharp
public record {DtoName}
{
    public {Type} {Property} { get; init; }  // {why this field}
    public {Type?} {NullableProperty} { get; init; }
}
```

---

## Database Changes

### Collection: `{collection-name}`

| Field | Type | Nullable | Reason |
|-------|------|----------|--------|
| `{fieldName}` | `{BsonType}` | Yes/No | {why} |

**New indexes:**

```javascript
// Reason: {why this index is needed}
db.{collection}.createIndex({ {field}: 1 }, { name: "{index-name}" })
```

If no DB changes: write "None."

---

## Backend Implementation

One block per file. Dependency order: Contracts → Accessors → Services → Controllers.

### {Layer}: `{ClassName}`

**File:** `{relative path from repo root}`
**Change:** Add / Modify / Delete
**Copied from:** `{pattern reference file}` *(omit if new from scratch)*

**New interface (if applicable):**
```csharp
public interface I{ClassName}
{
    Task<{ReturnType}> {MethodName}({ParamType} param, CancellationToken ct);
}
```

**Methods to add or change:**
```csharp
// {one-line description of what this does}
public async Task<{ReturnType}> {MethodName}({ParamType} param, CancellationToken ct)
```

**DI registration:** `[RegisterService]` on class / add to `Startup.cs` manually / no change

**Key rules for this file:**
- {convention rule the developer must not forget}

---

## Frontend Implementation

One block per file. Dependency order: service → component → module → routing.

### `{ClassName}` — {Component / Service / Module / Route}

**File:** `{relative path from repo root}`
**Change:** Add / Modify / Delete
**Copied from:** `{pattern reference file}` *(omit if new from scratch)*

**Key properties and methods:**
```typescript
// {description}
methodName(param: ParamType): Promise<ReturnType>

@Input() inputName: InputType;
@Output() outputName = new EventEmitter<EventType>();
```

**Template changes (if any):**
- Add `data-test-id="{id}"` to: {element description}
- Show/hide: {condition}
- Bind to: {service method}

**Module/routing wiring:**
- Declare in: `{module file}`
- Route: `{ path: '{path}', component: {Name}, canMatch: [...], data: { ... } }`

---

## Tests

### Unit Tests — {Layer}

**File:** `{relative path}`
**New file:** Yes / No
**Copied from:** `{pattern reference}`

**Mock setup:**
```csharp
private readonly Mock<I{Dep}> _{dep}Mock = new(MockBehavior.Strict);
// MockBehavior.Strict for controller tests; Loose for service tests
```

| Test name | Arrange | Assert |
|-----------|---------|--------|
| `{MethodName}_{Condition}_{Result}` | {what to mock/setup} | {what to verify} |

---

### Integration Tests

**File:** `{relative path}`
**New file:** Yes / No

**Seeder (if new data needed):**
- Class: `Seed{Entity}Data` in `DataSeed/`
- Insert: {what documents}
- Delete: {cleanup by ID}

| Test name | Request | Assert |
|-----------|---------|--------|
| `{test name}` | {HTTP method + endpoint + body} | {expected status + response shape} |

---

### Angular Spec Tests

**File:** `{relative path}`
**New file:** Yes / No

**TestBed providers:**
```typescript
mockService = jasmine.createSpyObj('{ServiceName}', ['{method1}', '{method2}']);
mockService.{method1}.and.returnValue(Promise.resolve({...}));
```

| Test name | `fakeAsync`? | Spy setup | Assert |
|-----------|-------------|-----------|--------|
| `should {behavior}` | Yes/No | `{method}.and.returnValue(...)` | {what to expect} |

---

## Implementation Sequence

Follow this order. Skip steps that do not apply.

1. Add/update DTOs and contracts in `Contracts/`
2. Update MongoDB document class in `Accessors/`
3. Add DB index (document in DbScripts)
4. Implement accessor method
5. Add/update service interface
6. Implement service method
7. Add/update controller action
8. Write unit tests for accessor
9. Write unit tests for service
10. Write unit tests for controller
11. Write integration test
12. Implement Angular service method
13. Implement Angular component changes
14. Write Angular spec tests
15. Run `dotnet test` — all green
16. Run `npm run test-headless` — all green

---

## NBS Rules Checklist

- [ ] Every controller action has `[Authorize]` + `[TenantSecuredFunction(...)]`
- [ ] No MongoDB access in Services — only via Accessors
- [ ] `[RegisterService]` on every new service/accessor class
- [ ] No `Version=` in `.csproj` — versions in `Directory.Packages.props` only
- [ ] New queried fields have indexes
- [ ] Sensitive data never stored plaintext — use `encryptedItemId`
- [ ] Angular components have `standalone: false`
- [ ] All new Angular HTTP calls use `lastValueFrom()`
- [ ] `httpMock.verify()` in every Angular spec `afterEach`
- [ ] `MockBehavior.Strict` for controller mocks, `Loose` for service mocks
- [ ] `NullLogger<T>.Instance` — never `new Mock<ILogger<T>>()`

---

## Assumptions

{Repeated here from above for developer visibility at end of plan.}

---

## Requirement Gaps

{AC items that are ambiguous, contradictory, or missing detail. If none: "None identified."}

---

## Notes for Developer

{Risk warnings, non-obvious constraints, or anything that will surprise a developer reading only this file.}
```
