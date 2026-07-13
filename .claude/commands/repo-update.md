Update the NBS workspace docs and `.gitignore` to register a new repo: `$ARGUMENTS`.

## Step 1 — Explore the new repo

Read `C:\neldevsrc\repos\$ARGUMENTS\` to understand the repo shape:
- List `src/` to identify layers and key modules
- Read `README.md` if present for purpose description
- Check for a `.sln` file name (may differ from the folder name)
- Note whether it has a `WebSpa/`, `WebComponent/`, Angular workspace, or is .NET-only

## Step 2 — Classify the repo

Decide which service group it belongs to:

| Group | Criteria |
|-------|----------|
| **Payment Hub Services** | Checkout, payment processing, banking, remittance, scheduled payments flows |
| **Platform/Infrastructure Services** | Auth, tenant, encryption, event bus, audit, menu, CSP |
| **Angular Frontend Libraries** | npm-only packages; no .NET WebApi |

## Step 3 — Update `architecture.md`

File: `.claude/rules/architecture/architecture.md`

Add a row to the correct service group table:
```
| `$ARGUMENTS` | <one-line role description> |
```

If the repo replaces an existing one, annotate with `(GitHub-migrated; replaces \`OldName/\`)` and update the old row or remove it.

## Step 4 — Update the microservice setup doc

**For Platform/Infrastructure:** `.claude/rules/microservice-setup/services-core.md`
**For Payment Hub:** `.claude/rules/microservice-setup/services-payment-hub.md`

Add a new `## <ServiceName> (\`$ARGUMENTS/\`)` section following the existing pattern:

```markdown
## <ServiceName> (`$ARGUMENTS/`)

**Purpose:** <what it does>

**Key src modules:**
- `ModuleName` — description
...

**Note:** <dependency or migration note if applicable>
```

If replacing an existing service, update that section's header and add a `**Migration:**` line.

## Step 5 — Update `backend.md` (backend .NET services only)

File: `.claude/rules/backend/backend.md`

- **Project responsibilities table** — add or update the row for this service
- **Responsibility boundaries table** — add a row if the service owns a distinct concern
- **Dependency diagram** — update if it's a platform-layer service
- **"Where to look first" table** — add a row if it introduces a new lookup pattern

Skip this step for Angular-only library repos.

## Step 6 — Update `dependencies.md`

File: `.claude/rules/microservice-setup/dependencies.md`

- If the repo publishes NuGet/npm packages consumed by others, update the **NuGet / npm Package Dependencies** bullet list
- If replacing an existing service, update the reference (e.g., `from Tenant` → `from \`xlr8-app-tenant\``)

## Step 7 — Update test tables

Update the per-repo rows in each test doc. Add a new row (or rename an existing one):

| File | Table |
|------|-------|
| `.claude/rules/tests/backend-service-test.md` | Per-Repo Notes |
| `.claude/rules/tests/integration-test.md` | Per-Repo Notes |
| `.claude/rules/tests/e2e-test.md` | Per-Repo Notes |

Use `—` for test projects that don't exist yet. Mirror the depth of similar services already in the table.

## Step 8 — Update `.gitignore`

File: `C:\neldevsrc\repos\.gitignore`

Add `$ARGUMENTS/` to the **Service repos** block in alphabetical order.

If the repo replaces an existing entry, keep the old entry (legacy repo may still exist locally) and add the new one.

## Rules

- Read each file before editing
- Do not remove existing entries unless explicitly told the old repo is gone
- Match formatting and style of existing entries exactly
- Do not add comments, docstrings, or extra prose beyond what existing entries have
