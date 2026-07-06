---
name: code-implementer
description: Rules for writing C# and Angular production code on the NBS platform
type: agent-skill
---

# Code Implementer

Rules for writing production code. Apply these without exception.

## Before Writing Any Code

1. Read the full implementation plan — internalize every step, file, and naming decision.
2. Read every file listed as "Pattern to follow" — the code must mirror that structure.
3. Read every file listed as "Files to modify" that already exists — never edit without reading.
4. Verify file paths exist (for modifications) or their parent directories exist (for new files). Flag anything missing before proceeding.

## Pre-flight Checks

If the plan includes contract changes affecting other services → flag explicitly:
> "This change affects shared contracts used by other services. Noted — continuing."

If the plan includes DB schema changes → flag explicitly:
> "This requires a DB migration script. Creating it as part of implementation."

## Execution Rules

For every implementation step:
1. Announce: what you are about to do + which file
2. Read the target file (if it exists)
3. Apply the change:
   - Match existing indentation, spacing, and code style exactly
   - Use exact names from the plan
   - Apply conventions from `nbs-conventions` skill
   - Add only what the plan specifies — no extras
4. Re-read the surrounding code after editing to verify structural integrity
5. Report: file path + what was added/changed

## Hard Rules — Never Break

- **No improvisation** — if the plan is ambiguous, resolve by looking at the reference pattern
- **No scope creep** — do not refactor surrounding code or fix unrelated issues
- **No `Version=`** in `.csproj` files — `Directory.Packages.props` owns versions
- **No inline ESLint disables** without a comment explaining why
- **Auth attributes** — every new API endpoint gets `[Authorize]` + `[SecuredFunction]`
- **i18n** — every new Angular user-facing string gets `i18n` attribute
- **NuGet packages** — if using a new framework feature (outbox, in-memory publishers), verify the package exists in both `Directory.Packages.props` AND the target `.csproj`. Missing package = "does not contain a definition" compiler error.

## CSP Messaging Patterns (CustomerServicePortal only)

- EventHub subscriber handlers: `[MessageHandlerOf("subscriber-id")]` — must have matching entry in `appsettings.json` `EventHubMessaging.Subscribers`. Missing config crashes startup.
- Outbox/in-memory consumer handlers: `[RegisterService]` **only** — never `[MessageHandlerOf]`, never `[RegisterBackgroundService]`.
- Startup messaging chain (all five, in this order): `.WithEventHub(Configuration).WithInMemoryPublishers(Configuration).WithMongoDbOutbox(Configuration).RunSubscribers().InitializePublishers()`
- Required packages: `Nbs.Framework.Messaging.InMemory`, `Nbs.Framework.Messaging.Outbox.MongoDb`

## Layer Discipline

```
WebApi → Services → Accessors → external
```

- Accessor: never reference Service or WebApi
- Service: never reference WebApi
- DTOs/contracts: live in `Contracts/` — never in `Services/` or `WebApi/`

## After All Steps

Report:

```
IMPLEMENTATION COMPLETE
Files created:
- {path} — {what it contains}

Files modified:
- {path} — {what changed}
```
