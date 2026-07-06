---
name: ado-story-reader
description: Fetch an ADO work item and extract all fields needed for analysis or review
type: agent-skill
---

# ADO Story Reader

Fetch a work item from Azure DevOps and extract everything downstream agents need.

## Connection

- **Org:** `nbsdev` | **Project:** `Services`
- **PAT location:** `C:\neldevsrc\repos\.mcp.json` → field `AZURE_DEVOPS_EXT_PAT`

## Primary Path — MCP

Call `mcp__azure-devops__wit_get_work_item` with:
- `id`: the story/work-item ID
- `project`: `Services`
- `expand`: `relations`

In parallel, if commit or PR context is needed:
- `mcp__azure-devops__repo_list_pull_requests_by_commits`
- `mcp__azure-devops__repo_search_commits`

## Fallback — REST API

When MCP tools are unavailable:

```bash
PAT="<value from .mcp.json>" && \
curl -s -u ":$PAT" \
  "https://dev.azure.com/nbsdev/Services/_apis/wit/workitems/{id}?api-version=7.1&\$expand=all"
```

For linked relations:
```bash
curl -s -u ":$PAT" \
  "https://dev.azure.com/nbsdev/Services/_apis/wit/workitems/{id}/relations?api-version=7.1"
```

## Fields to Extract

Always extract:
- `title` — exact verbatim title (used as Draft PR title — preserve exactly)
- `description`
- `acceptanceCriteria` (field: `Microsoft.VSTS.Common.AcceptanceCriteria`)
- `state`
- `assignedTo`
- `iterationPath`
- `areaPath`
- `tags`
- `relations` — linked PRs, commits, child work items, test cases

## Output Contract

Return a structured object the calling agent can reference:

```
STORY FETCHED
ID: {id}
Title: {exact verbatim title}
State: {state}
Assigned: {assignedTo}
Iteration: {iterationPath}
Area: {areaPath}
Tags: {tags}

Description:
{description}

Acceptance Criteria:
{acceptanceCriteria — verbatim, numbered}

Linked Items:
- PRs: {list or none}
- Commits: {list or none}
- Test Cases: {list or none}
- Child items: {list or none}
```
