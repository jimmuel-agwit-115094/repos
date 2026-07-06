---
name: draft-pr-creator
description: Create a DRAFT pull request on ADO or GitHub with the exact ADO story title
type: agent-skill
---

# Draft PR Creator

Create a DRAFT pull request after implementation is complete. Never create a non-draft PR.

## Rules

- PR title = **exact verbatim ADO story title** — do not paraphrase, truncate, or modify
- Always create as **DRAFT** — abort if the target system does not support draft PRs
- Target branch = `main` (detect via `git branch -r | grep main`; fallback to `master`)
- Source branch = current git branch (`git branch --show-current`)
- Link the ADO work item to the PR after creation

## Step 1 — Detect Current Branch

```bash
git branch --show-current
```

If detached HEAD or no commits: stop and report error — cannot create PR without a branch.

## Step 2 — Detect Target Branch

```bash
git branch -r | grep -E "origin/(main|master)" | head -1
```

Use `main` if found, else `master`. If neither found, report and ask before proceeding.

## Step 3 — Push Branch (if needed)

```bash
git push -u origin {current-branch}
```

If push fails (permissions, no remote): report the error and stop.

## Step 4 — Create Draft PR

### Primary: ADO

Call `mcp__azure-devops__repo_create_pull_request` with:
- `repositoryId`: detect from `git remote get-url origin` (extract repo name)
- `project`: `Services`
- `title`: exact ADO story title (from `ado-story-reader` output — field `Title`)
- `sourceRefName`: `refs/heads/{current-branch}`
- `targetRefName`: `refs/heads/{target-branch}`
- `isDraft`: `true`
- `description`: `Work item: #{story-id}`

### Fallback: GitHub

If not an ADO repo (remote URL contains `github.com`):

```bash
gh pr create \
  --title "{exact ADO story title}" \
  --body "Work item: #{story-id}" \
  --draft \
  --base {target-branch}
```

## Step 5 — Link Work Item to PR

After PR is created in ADO, call `mcp__azure-devops__wit_link_work_item_to_pull_request` with:
- `workItemId`: the story ID
- `pullRequestId`: the PR ID returned from Step 4
- `repositoryId`: repo name
- `project`: `Services`

## Output

Return:

```
DRAFT PR CREATED
PR Title: {exact title}
PR URL: {url}
PR ID: {id}
Source: {current-branch} → {target-branch}
Work item linked: #{story-id}
```

## Abort Conditions

- Current branch is `main` or `master` — never create a PR from a protected branch
- No commits ahead of target — nothing to review
- Draft flag not supported — report and stop; do not create non-draft
