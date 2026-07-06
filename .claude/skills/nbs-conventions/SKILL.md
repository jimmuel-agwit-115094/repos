---
name: nbs-conventions
description: Which NBS convention docs to load and the key rules every agent must follow
type: agent-skill
---

# NBS Conventions

Every agent working on NBS code must load these docs before writing or reviewing anything. Do not rely on memory — read them fresh each time.

## Convention Docs — Always Load

| File | Governs |
|------|---------|
| `C:\neldevsrc\repos\.claude\architecture.md` | Service groups, solution layout, hosting, auth, feature flags |
| `C:\neldevsrc\repos\.claude\conventions.md` | C#, Angular/TS, CI/CD, DB naming and patterns |
| `C:\neldevsrc\repos\.claude\dependencies.md` | Inter-service dependency graph, local startup order |

## Convention Docs — Load When Relevant

| File | Load when |
|------|-----------|
| `C:\neldevsrc\repos\.claude\backend-service-test.md` | Any C# code or tests are being written/reviewed |
| `C:\neldevsrc\repos\.claude\integration-test.md` | Any integration tests are in scope |
| `C:\neldevsrc\repos\.claude\spa-service-test.md` | Any Angular SPA changes are in scope |
| `C:\neldevsrc\repos\.claude\e2e-test.md` | Any E2E tests are in scope |
| `{repo}/CLAUDE.md` | The affected repo has its own CLAUDE.md (e.g. `CustomerServicePortal/CLAUDE.md`) |

## Non-Negotiable Rules (Memorize These)

### C# / .NET
- Layer direction: `WebApi → Services → Accessors → external`. No upward refs.
- No inline `Version=` on `<PackageReference>` — `Directory.Packages.props` owns all versions.
- DTOs: PascalCase, suffixed `Request` / `Response` / `Event`. Live in `Contracts/` project.
- Events: past-tense names (`PaymentProcessed`, `OrderCreated`).
- Every new API endpoint: `[Authorize]` + `[SecuredFunction]` attributes required.
- Project naming: `{Repo}.{Layer}` pattern.

### Angular / TypeScript
- Component selectors: `kebab-case`.
- Class names: `PascalCase`.
- Routes: `/web/{service-name}/` base URL.
- Every new user-facing string: `i18n` attribute required. Three locales: `en`, `es`, `eo`.
- After adding i18n strings: `npm run i18n:extract` must be run.

### CSP-Specific (CustomerServicePortal only)
- Outbox consumer handlers: `[RegisterService]` only. Never `[MessageHandlerOf]` or `[RegisterBackgroundService]` for outbox consumers.
- EventHub subscriber handlers: `[MessageHandlerOf("subscriber-id")]` with matching `appsettings.json` entry.
- Messaging startup chain (all five, in order): `.WithEventHub(...).WithInMemoryPublishers(...).WithMongoDbOutbox(...).RunSubscribers().InitializePublishers()`
- Outbox NuGet packages: `Nbs.Framework.Messaging.InMemory` + `Nbs.Framework.Messaging.Outbox.MongoDb`.
