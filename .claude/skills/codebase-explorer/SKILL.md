---
name: codebase-explorer
description: Systematic strategy for exploring the NBS codebase to find reference patterns and affected files
type: agent-skill
---

# Codebase Explorer

Systematic search strategy for finding real evidence before designing or implementing anything. Never guess — grep and read.

## Root

All repos live under `C:\neldevsrc\repos`. Each subfolder is an independent repo.

## Step 1 — Search by Domain Keywords

Take the domain keywords from the story brief. For each keyword:

```
Grep for keyword across C:\neldevsrc\repos
- Search *.cs, *.ts, *.html files
- Exclude: bin/, obj/, node_modules/, .git/
```

Record every hit with file path + line number. Cluster by repo and layer.

## Step 2 — Find the Reference Pattern

The reference pattern = the most similar existing feature. This becomes the implementation template.

What to look for:
- A method in `Services/` that does something structurally similar
- A controller action at the same HTTP verb/route pattern
- An accessor that queries the same or similar collection
- An Angular component with the same data-fetch + display pattern

Read 30–50 lines around the best match. This is your primary template — mirror it, do not invent.

## Step 3 — Map Affected Layers

For each layer the story implies changing, find the existing file:

| Layer | Where to look |
|-------|--------------|
| Service | `src/Services/{Domain}Service.cs` |
| Accessor | `src/Accessors/{Domain}Accessor.cs` |
| Controller | `src/WebApi/Controllers/{Domain}Controller.cs` |
| DTO | `src/Contracts/{Domain}Request.cs`, `{Domain}Response.cs` |
| Angular component | `src/{workspace}/src/app/{feature}/` |

Read the relevant section (class declaration, method signatures, 10–20 lines of context).

## Step 4 — Check MongoDB

If data is involved:
- Find the accessor class — read the collection name and any index definitions
- Check `DbScripts/` for existing migration script patterns

## Step 5 — Check Cross-Service

If the story brief flagged cross-service signals:
- Grep for `IEventPublisher`, the event class name
- Grep for consumer background service or message handler
- Check `Client.Contracts/` for shared DTOs

## Step 6 — Check Feature Flags

If the story implies gradual rollout:
- Grep for `IFeatureFlags`, `GetVariationAsync`
- Find the nearest existing flag usage as the pattern

## Step 7 — Find Existing Tests to Update

For each class or method that will change:
- Grep its name across `*.UnitTests`, `*.IntegrationTests`, `*.spec.ts`
- Record: file path, test class, test method name, why it will break

## Step 8 — Lock In Test Patterns

Read 2–3 existing test classes in the affected test project. Extract:
- Class declaration + constructor (DI wiring pattern)
- `Mock<T>` declaration and setup style
- Arrange/Act/Assert layout and blank line usage
- Assertion style (`Should().Be(...)` etc.)
- Method naming: `MethodName_Condition_ExpectedResult`
- Any shared fixtures, base classes, or helpers

All new tests must match this style exactly.
