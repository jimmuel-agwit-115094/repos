---
name: pr-commenter
description: Format and post review findings as PR comment threads on ADO
type: agent-skill
---

# PR Commenter

Post review findings as structured comment threads on an ADO pull request.

## Comment Format

Each posted comment:

```
**[{SEVERITY}]** {Problem description — one sentence, precise}

**Fix:** {Specific action to take — concrete enough to implement without research}

{Optional: 1–2 lines of context if the fix isn't self-evident}
```

No preamble. No hedging. No "you might want to consider...". Problem, fix, done.

## Overall Assessment Comment Format

Post this as a general (non-inline) thread first:

```
## PR Review Summary

**Verdict:** {APPROVE / NEEDS CHANGES / CRITICAL ISSUES}

{1–3 sentences: what the PR does, overall quality, top concern.}

**Stats:** {X} critical, {Y} major, {Z} minor, {W} notes

Checklist (critical + major):
- [ ] #{n}: {one-line summary} ({file}:{line})
```

## Posting Inline Comments

For file-specific findings, call `mcp__azure-devops__repo_create_pull_request_thread` with:
- `repositoryId`: `{repoName}`
- `pullRequestId`: `{prId}`
- `project`: `Services`
- `filePath`: `{filePath from finding}`
- `rightFileStartLine`: `{line_start}`
- `rightFileEndLine`: `{line_end}`
- `rightFileStartOffset`: `1`
- `rightFileEndOffset`: `1`
- `status`: `Active`
- `content`: formatted comment (above)

## Posting General Comments

For findings not tied to a specific file (overall assessment, cross-cutting risks):
- Same call as above but omit `filePath` and all line parameters.

## Posting Order

1. Overall assessment first (general thread)
2. Then inline file-level findings, ordered by severity (CRITICAL → MAJOR → MINOR → NOTE)

## After Posting

Report:

```
COMMENTS POSTED
PR: #{prId} in {repoName}
Posted: {N} total
- {count} inline file comments
- 1 overall assessment comment
PR URL: https://dev.azure.com/nbsdev/Services/_git/{repoName}/pullrequest/{prId}
```
