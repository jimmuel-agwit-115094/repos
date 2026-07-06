---
name: pr-diff-reader
description: Fetch ADO pull request metadata, full diff, and file content for code review
type: agent-skill
---

# PR Diff Reader

Fetch everything needed to review a pull request: metadata, full diff, and file content.

## Step 1 — Parse Input

Input can be any of:
| Format | Example |
|--------|---------|
| Full ADO URL | `https://dev.azure.com/nbsdev/Services/_git/Banking/pullrequest/123456` |
| Repo + PR ID | `Banking/123456` or `Banking 123456` |
| PR ID only | `123456` (repo name needed — ask if cannot infer) |

Extract:
- `repoName` — e.g. `Banking`, `CustomerServicePortal`, `Payments`
- `prId` — numeric PR ID
- `project` — always `Services`

## Step 2 — Fetch PR Metadata

Call `mcp__azure-devops__repo_get_pull_request_by_id` with:
- `repositoryId`: `{repoName}`
- `pullRequestId`: `{prId}`
- `project`: `Services`
- `includeChangedFiles`: `true`
- `includeWorkItemRefs`: `true`

Record: title, description, author, source branch, target branch, linked work items, status, isDraft.

## Step 3 — Fetch Full Diff

Call `mcp__azure-devops__repo_get_pull_request_changes` with:
- `repositoryId`: `{repoName}`
- `pullRequestId`: `{prId}`
- `project`: `Services`
- `includeDiffs`: `true`
- `includeLineContent`: `true`
- `top`: `100`

Record every changed file, change type (add/edit/delete), and line-by-line diff.

If more than 100 files: paginate with `skip` until all changes fetched.
Flag: PRs with 50+ changed files are a risk signal ("Large PR — consider splitting").

## Step 4 — Fetch Full File Content

For each modified file (not deleted), call `mcp__azure-devops__repo_get_file_content` with:
- `repositoryId`: `{repoName}`
- `path`: `{filePath}`
- `project`: `Services`
- `version`: `{sourceBranch}`
- `versionType`: `Branch`

**Fetch strategically:** only `.cs`, `.ts`, `.html`, `.json`, `.csproj`, `.yaml` files with substantive logic changes. Skip generated files, lock files, binaries.

## Output Contract

Return structured PR data:

```
PR FETCHED
PR: #{prId} — {title}
Author: {author}
Branch: {sourceBranch} → {targetBranch}
Files changed: {count}
Linked work items: {list or "None"}
Status: {status} | Draft: {true/false}

Changed files:
- {filePath} [{add/edit/delete}]
  Diff: {line-by-line changes}
  Full content: {if fetched}
```
