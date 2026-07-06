# MCP — Azure DevOps

Configured via `.mcp.json` at workspace root. Uses `azure-devops-mcp` (read-only).

## Connection Details

| Setting | Value |
|---------|-------|
| Organization | `nbsdev` |
| Project | `Services` |
| URL | `https://dev.azure.com/nbsdev` |
| Package | `azure-devops-mcp@latest` (npx, no install needed) |

## PAT Setup

PAT stored directly in `.mcp.json`. No env var needed.

To regenerate: ADO → profile → **Personal access tokens** → new token with scopes:
- **Work Items**: Read
- **Code**: Read
- **Build**: Read
- **Release**: Read
- **Test Management**: Read

Update `AZURE_DEVOPS_PAT` value in `.mcp.json` and restart Claude Code.

## Available Tools (via MCP)

Once connected, Claude can use:

| Category | What you can ask |
|----------|-----------------|
| Work Items | "Get user story #12345", "List open bugs in sprint", "Find work items for feature X" |
| Git / PRs | "Show PR #114100 in xlr8PageTemplate", "List open PRs", "Get diff for PR" |
| Builds | "Show latest build for Banking", "Any failed builds today?" |
| Pipelines | "List pipelines for Checkout", "Show last pipeline run" |
| Projects | "List all projects", "Get project details for Services" |

## Verify Connection

In Claude Code, ask:

> "Test the ADO connection and list projects"

Claude will call `test_connection` → `list_projects` via the MCP server.

## Notes

- **Read-only** — cannot create/update work items or PRs via this MCP
- PAT stored as env var, NOT in `.mcp.json` (`.mcp.json` is committed to source control)
- If PAT expires, regenerate in ADO and update the env var
