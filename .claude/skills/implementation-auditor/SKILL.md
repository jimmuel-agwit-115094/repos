---
name: implementation-auditor
description: Full audit checklist for verifying developer output against the implementation plan
type: agent-skill
---

# Implementation Auditor

Verify what was actually written against what the plan required. Read every file. Do not assume the developer followed the plan.

## Severity Levels

| Level | Meaning |
|-------|---------|
| `BLOCKER` | Ship-stopping. Missing file, missing test, no assertion, compile-breaking issue. Must fix. |
| `MAJOR` | Significant gap. Wrong behavior, wrong mock, untested AC scenario. Fix before ship unless waived. |
| `MINOR` | Style or naming deviation. Does not affect correctness. |
| `NOTE` | Informational. No action required unless context changes. |

## Audit 1 — Implementation vs Plan

For each file in the plan's affected layers:
1. Verify the file exists — if "Files to create" entry is missing: `BLOCKER: file not created`
2. Read the file — verify exact classes, methods, properties, attributes the plan specified
3. Cross-check naming — class, method, DTO, event names must match the plan exactly
4. Check layer discipline — no upward dependencies (`Accessor` must not reference `Service`, etc.)
5. Check for scope creep — flag any class/method/property not in the plan
6. Check CSP rules (if `CustomerServicePortal`):
   - Outbox handlers: `[RegisterService]` only
   - EventHub handlers: `[MessageHandlerOf("id")]` with matching `appsettings.json` entry
   - Messaging chain order: `.WithEventHub(...).WithInMemoryPublishers(...).WithMongoDbOutbox(...).RunSubscribers().InitializePublishers()`
7. Check `appsettings.json` — verify new settings sections exist and match code
8. Check `.csproj` — no inline `Version=` on any `<PackageReference>`
9. Check auth — every new API endpoint has `[Authorize]` + `[SecuredFunction]`
10. Check i18n — every new Angular user-facing string has `i18n` attribute

## Audit 2 — Unit Tests

For each test in the plan's "Tests to update" (Unit):
- Find the file → find the method → read full body
- Verify the fix was applied per "What to change" column
- Verify test intent is preserved (a fix that deletes the assertion = `BLOCKER`)
- Check using directives: removed using still has consumers = compile error = `BLOCKER`

For each test in "Tests to write" (Unit):
| Check | Pass condition | Severity |
|-------|---------------|----------|
| Method exists | Present in file | BLOCKER |
| Full body | No `// TODO`, no empty Act/Assert | BLOCKER |
| Real Arrange values | No `"test"`, `0`, `null` unless testing null | MAJOR |
| Assertion present | At least one `Should()` call | BLOCKER |
| Asserts right thing | Assertion targets property from plan's "Scenario" | MAJOR |
| No phantom pass | Not `true.Should().BeTrue()` | BLOCKER |
| Naming | `MethodName_Condition_ExpectedResult` | MINOR |
| AAA structure | Three sections, blank line between each | MINOR |
| Mock setup complete | Every dependency called in Act is set up in Arrange | MAJOR |
| Exception tests | Use `Func<Task> act = ...; await act.Should().ThrowAsync<T>()` | MINOR |
| Time-dependent | Uses `Nbs.Framework.TimeTravel` before Act | MAJOR |

## Audit 3 — Integration Tests

For each integration test in the plan:
| Check | Pass condition | Severity |
|-------|---------------|----------|
| Method exists | Present | BLOCKER |
| Full body | No stubs | BLOCKER |
| `WebServer` helper | Not raw `WebApplicationFactory` | MAJOR |
| Full HTTP request | Method, path, headers, body | MAJOR |
| Full response assertion | Status code AND body assertions | MAJOR |
| No hardcoded connections | Uses `appsettings.IntegrationTests.json` | MAJOR |
| `appsettings.IntegrationTests.json` | New config sections from plan exist | BLOCKER |

## Audit 4 — SPA Tests (Angular)

Only if plan includes SPA changes:
| Check | Pass condition | Severity |
|-------|---------------|----------|
| Spec file exists | Present | BLOCKER |
| `it(...)` block exists | Present | BLOCKER |
| Real `expect()` | Not `expect(true).toBe(true)` | BLOCKER |
| `TestBed` mirrors existing | Same providers, imports | MAJOR |
| Async handled | `fakeAsync`, `async/await`, or `done` | MAJOR |

## Audit 5 — ADO Test Cases

Fetch test cases linked to the story (`Tested By` relations). For each:
- `COVERED` — implementation satisfies all steps
- `UNCOVERED` — missing behavior → `MAJOR`
- `PARTIALLY COVERED` — some steps satisfied → `MAJOR`
- `OUT OF SCOPE` — outside this story's AC → `NOTE`
- `INVALID` — nonsensical, duplicated, or contradicts AC → `NOTE`

## Output Format

Return a structured verdict:

```
QA VERDICT
==========
Verdict: SHIP / BLOCK / CONDITIONAL SHIP

Implementation Findings:
| Severity | File | Finding |

Unit Test Findings:
| Severity | Test File | Test Method | Finding |

Integration Test Findings:
| Severity | Test File | Test Method | Finding |

SPA Test Findings:
| Severity | Spec File | Test Block | Finding |

ADO Test Case Findings:
| ADO ID | Title | Outcome | Severity | Notes |

Required Actions Before Ship:
1. {BLOCKER/MAJOR items in priority order}

"None — ready to ship." if SHIP.
```
