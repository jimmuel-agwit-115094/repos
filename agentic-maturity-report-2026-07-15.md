# Agentic Maturity Report

**Date:** 2026-07-15
**Developer:** Agwit, Jimmuel
**Data window:** 2026-04-14 → 2026-07-15 (92 days, 61 active days)

---

## 1. Headline

### Stage 4 — Orchestration-Ready

This developer has authored and routinely operates a composable multi-agent pipeline (BA → Architect → Developer → Reviewer → PR Reviewer) that chains cross-domain stages, delegates scoped work to parallel subagents, self-verifies via build/test execution, and integrates results through artifact hand-offs — meeting the full Stage 4 definition.

---

## 2. Stage Ladder

| Stage | Name | Demonstrated? | Evidence |
|-------|------|---------------|----------|
| **0** | Assistive | routine | Text-only Q&A present in many sessions (quick questions, explanations) — normal baseline |
| **1** | Guided Task | routine | Single-tool sessions exist (Read-only exploration, one-off Bash commands) |
| **2** | Multi-Step | routine | Multi-tool edit sessions: Read → Grep → Edit → Read cycles across files |
| **3** | Autonomous Workflow | routine | 844-turn and 782-turn sessions: end-to-end feature implementation and test-fixing with auto mode (`skipAutoPermissionPrompt: true`), Claude runs builds/tests, commits, creates PRs autonomously |
| **4** | **Orchestration-Ready** | **routine** | 5-agent authored pipeline, 128 Agent invocations, 15,069 subagent turns, parallel developer agents, cross-domain ADO MCP integration, 6 practiced agent types |

**Ceiling: Stage 4** — the highest stage routinely demonstrated.

---

## 3. Why This Stage, Not the Next

**Why Stage 4 (not higher):** Stage 4 is the highest defined stage. To deepen orchestration further, the developer could:
- Add **scheduled/cron-based agents** for automated CI monitoring or story triage (none configured)
- Implement **verification hooks** that enforce quality gates automatically (no hooks configured)
- Enable **cross-domain MCP composition** in single flows (ADO + Datadog + browser automation together — Datadog and Playwright are configured but only Trial/unused)
- Build **feedback loops** where reviewer findings automatically feed back to developer agents for correction without human intervention

---

## 4. Capability Utilization (Practice, Not Possession)

### Installed vs Practiced

| Category | Installed | Practiced | Utilization |
|----------|-----------|-----------|-------------|
| Plugins | 2 | 2 (caveman, claude-mem) | 100% |
| Skills (project) | 4 | 1 (story-getter) | 25% |
| Commands (project) | 6 | 5 (analyst, architect, developer, pr-reviewer, reviewer) | 83% |
| Agents (project) | 5 | 6 practiced types* | — |
| MCP servers | 3 | 1 (azure-devops) | 33% |

\* Agent types include both project-defined and built-in (Explore). Some older command names (/business-analyst, /developer, /architect) map to the same pipeline stages as current nbs-* agents.

### Practiced Capabilities

| Capability | Type | Invocations | Sessions | Last Used | Status |
|------------|------|-------------|----------|-----------|--------|
| `story-getter` | Skill | 24 | 24 | 2026-07-13 | PRACTICED |
| `Explore` | Agent | 44 | 19 | 2026-07-13 | PRACTICED |
| `analyst` | Agent | 11 | 9 | 2026-07-13 | PRACTICED |
| `nbs-analyst` | Agent | 8 | 5 | 2026-07-13 | PRACTICED |
| `developer` | Agent | 5 | 4 | 2026-07-13 | PRACTICED |
| `architect` | Agent | 4 | 4 | 2026-07-13 | PRACTICED |
| `reviewer` | Agent | 3 | 2 | 2026-07-13 | PRACTICED |
| `azure-devops` | MCP | 273 | 43 | 2026-07-15 | PRACTICED |
| `/business-analyst` | Command | 21 | 19 | 2026-07-01 | PRACTICED |
| `/developer` | Command | 16 | 15 | 2026-07-13 | PRACTICED |
| `/architect` | Command | 14 | 13 | 2026-07-13 | PRACTICED |
| `/nbs-analyst` | Command | 8 | 5 | 2026-07-13 | PRACTICED |
| `/review-pr` | Command | 6 | 4 | 2026-07-06 | PRACTICED |
| `/craft` | Command | 3 | 2 | 2026-07-06 | PRACTICED |

### Trial (invoked but below threshold)

| Capability | Invocations | Sessions | Notes |
|------------|-------------|----------|-------|
| `implementation-plan` (skill) | 1 | 1 | Used once for story 862249 |
| `nbs-pr-reviewer` (agent) | 2 | 2 | Recent (2026-07-15), approaching practiced |
| `nbs-architect` (agent) | 1 | 1 | Recent — renamed from `architect` |
| `nbs-developer` (agent) | 1 | 1 | Recent — renamed from `developer` |
| `nbs-reviewer` (agent) | 1 | 1 | Recent — renamed from `reviewer` |
| `playwright` (MCP) | 3 | 1 | Single session (2026-06-29) |

### Stale (practiced once but >60 days)

| Capability | Last Used | Notes |
|------------|-----------|-------|
| `/agents` | 2026-05-06 | Exploration command |
| `/upgrade-angular-20` | 2026-05-06 | One-time migration |
| `/caveman:caveman` | 2026-04-16 | Invoked via command; plugin still active via startup hook |
| `/statusline` | 2026-05-12 | Configuration command |

### Dormant (installed, never invoked)

| Capability | Type |
|------------|------|
| `datadog` | MCP server (home-level) |
| `datadog-mcp` | MCP server (project-level) |
| `test-fixer` | Skill (project) — invoked once as `/test-fixer` (TRIAL) |
| `ui-fixer` | Skill (project) — invoked twice as `/ui-fixer` (TRIAL) |

**Overall utilization note:** The developer actively practices the core pipeline capabilities (analyst → architect → developer → reviewer → PR reviewer) and the Azure DevOps MCP integration. Datadog MCP is configured at two levels but never invoked — a clear dormant finding. The nbs-* agent names are recent renames of practiced capabilities (/business-analyst → /analyst → /nbs-analyst), so the low nbs-* invocation counts understate actual practice.

---

## 5. Supporting Dimension Scores

| Dim | Score | Justification |
|-----|-------|---------------|
| **D1 Task scope** | 3 | End-to-end jobs routine: 844-turn implementation session, 782-turn test-fix marathon, story-to-PR pipelines. Avg 57.2 turns/session. TaskCreate/TaskUpdate used (35 calls). Plan mode used (11 ExitPlanMode calls). |
| **D2 Tool use** | 3 | 32 distinct tools. Authored skills (4), commands (6), agents (5), plugins (2). MCP: Azure DevOps (273 inv, PRACTICED). Composable tooling is the norm, not the exception. |
| **D3 Autonomy granted** | 3 | `skipAutoPermissionPrompt: true` — auto mode default. User gives goals ("Process ADO story 862249"), Claude runs full pipeline. Caveman mode active most sessions. Few interjections per job. |
| **D4 Self-verification** | 2 | Developer agents run builds/tests/lint (StyleCop) to verify work. Story 862249: 329 unit tests verified, build success confirmed. But no verification hooks configured; some sessions skip verification entirely. Not yet systematic across all workflows. |
| **D5 Orchestration & safety** | 3 | Parallel multi-agent (2 developer agents in parallel), authored 5-stage pipeline, cross-domain MCP (ADO work items → code → PRs), plugin ecosystem (caveman, claude-mem). No cron/scheduled agents yet. |

---

## 6. Evidence

### Evidence A: Full Pipeline Orchestration (Stage 4 defining)

**Session:** `fac6b23e` (story 862249 — tenant hierarchy lock API)

User prompt (single directive):
```
Process ADO story 862249. Fetch the story, analyze requirements, explore the codebase,
and write a full implementation plan to .claude/implementation-plan/862249.md
```

Architect agent invokes story-getter skill:
```json
{"type":"tool_use", "name":"Skill", "input":{"skill":"story-getter","args":"862249"}}
```

Two developer agents spawned in parallel (330 combined subagent turns). Developer self-verification mid-execution:
```
"329 tests pass (was 324, added 5 new `LockHierarchy` tests).
Now build the integration test project."
```

**Pattern:** One user directive → Skill (story-getter) → Agent (architect) → 2× Agent (developer, parallel) → build/test verification → complete. Zero user interjections.

### Evidence B: Autonomous Test-Fixing (Stage 3)

**Session:** `7ee65a1a` (782 turns — Angular Karma spec fixing)

User supplied error logs directly:
```
ERROR: 'Error searching tenants', HttpErrorResponse{...}
Chrome Headless 127.0.6533.88 (Windows 10) SearchTenantsComponent full access user Searching
should re-apply doesFilterPass override on filterChanged event FAILED
    TypeError: Cannot read properties of undefined (reading 'length')
        at ClientSideRowModel.onFilterChanged
```

Claude autonomously diagnosed root cause (fdescribe leak + AG Grid API misuse), applied surgical fixes:
1. `Read search-tenants.component.spec.ts offset=46` → sees `fdescribe`
2. `Edit fdescribe → describe` → removes focus blocker
3. `Read offset=600` → inspects grid test
4. `Edit` → replaces `dispatchEvent` with proper AG Grid filter API + adds error spy

User interjected 3–4 times only to supply fresh error logs as course corrections. Claude drove all diagnosis and fixes.

### Evidence C: PR Review via MCP Pipeline (Stage 4)

**Session:** `d22e346e` (Menu PR review)

User prompt:
```
/nbs-pr-reviewer https://dev.azure.com/nbsdev/Services/_git/6e016b7e-d591-4f07-938e-af47555d68e1/pullrequest/118961
```

Agent delegation:
```json
{"type":"tool_use", "name":"Agent",
 "input":{"subagent_type":"nbs-pr-reviewer",
  "prompt":"Review the pull request at https://dev.azure.com/...pullrequest/118961\nFetch PR details, review all changes against NBS standards, and post findings as comments on the PR."}}
```

Subagent autonomously:
1. `mcp__azure-devops__repo_get_pull_request_by_id` → fetched PR metadata
2. `mcp__azure-devops__repo_get_pull_request_changes` → retrieved diffs for all 6 files
3. Reviewed each file against NBS conventions (layers, naming, auth attributes)
4. `mcp__azure-devops__repo_create_pull_request_thread` → posted summary verdict (Thread ID: 1427298)

Result: Approved (0 issues). Entire flow from user prompt to ADO comment — zero intervention.

### Evidence D: Surgical Edits with Caveman Mode (Stage 2–3)

**Session:** `653b536a` (recent, short)

User prompt:
```
Verify each finding against current code. Fix only still-valid issues, skip the
rest with a brief reason, keep changes minimal, and validate.

In `@test/Services.UnitTests/MessageHandlers/PersonChangedHanderTests.cs` around
lines 101 - 107, Update the Person fixtures...
```

**Pattern:** Goal-oriented with specifics. User trusts Claude to verify validity, skip non-issues, and apply minimal changes. Caveman mode active — terse responses, no filler.

---

## 7. Usage Profile

| Metric | Value |
|--------|-------|
| **Data window** | 2026-04-14 → 2026-07-15 (92 calendar days) |
| **Active days** | 61 |
| **Sessions analyzed** | 285 |
| **User prompts** | 1,289 |
| **Avg prompt length** | 124.7 chars |
| **Assistant turns** | 16,299 |
| **Avg turns/session** | 57.2 |
| **Tool-using turn %** | 63.2% |
| **Tool diversity** | 32 distinct tools |
| **Subagent (sidechain) turns** | 15,069 (92.5% of all assistant turns) |
| **Plan/track calls** | 35 (TaskCreate + TaskUpdate + TodoWrite) |
| **Plan mode exits** | 11 (ExitPlanMode) |
| **Output tokens total** | 10,448,060 |
| **Distinct projects** | 14 |
| **MCP tools used** | 18 distinct Azure DevOps + Playwright tools |
| **Permission mode** | Auto (`skipAutoPermissionPrompt: true`) |
| **Model preference** | `opus[1m]` (configured default) |

### Top Tools

| Tool | Count |
|------|-------|
| Read | 4,161 |
| Bash | 2,716 |
| Grep | 1,018 |
| Edit | 961 |
| Glob | 707 |
| Write | 157 |
| Agent | 128 |
| mcp__azure-devops__wit_get_work_item | 102 |
| ToolSearch | 97 |
| mcp__azure-devops__repo_get_file_content | 51 |
| mcp__azure-devops__repo_create_pull_request_thread | 28 |
| Skill | 26 |

### Models Used

| Model | Turns |
|-------|-------|
| claude-sonnet-4-6 | 7,124 (43.7%) |
| claude-opus-4-6 | 4,736 (29.1%) |
| claude-haiku-4-5 | 4,428 (27.2%) |

### Top Practiced Commands

| Command | Invocations | Sessions |
|---------|-------------|----------|
| /resume | 51 | 45 |
| /exit | 35 | 34 |
| /compact | 28 | 16 |
| /business-analyst | 21 | 19 |
| /developer | 16 | 15 |
| /architect | 14 | 13 |
| /model | 14 | 8 |
| /analyst | 10 | 9 |
| /nbs-analyst | 8 | 5 |
| /rename | 7 | 5 |
| /review-pr | 6 | 4 |

### Longest Sessions

| Session | Turns | Tool Calls | Subagent Turns |
|---------|-------|------------|----------------|
| 5e2608c2 | 844 | 456 | 0 |
| 7ee65a1a | 782 | 423 | 0 |
| 12109c70 | 489 | 262 | 0 |
| 5f89fc3b | 393 | 215 | 0 |
| cd69deff | 347 | 183 | 0 |
| agent-a11b5a34 | 195 | 131 | 330 |

---

## 8. Level-Up Recommendations

Already at Stage 4. Three moves to deepen orchestration maturity:

### 1. Activate or Remove Dormant MCP Servers

Datadog MCP is configured at **two levels** (home `settings.json` and project `.mcp.json`) but has **zero invocations**. Either:
- Integrate Datadog into the reviewer/PR-reviewer pipeline (check APM impact of changes, verify no new error spikes post-deploy)
- Remove the configuration to reduce noise

Similarly, Playwright MCP (3 invocations, 1 session = TRIAL) could be wired into the `ui-fixer` skill for automated visual verification.

### 2. Add Verification Hooks

No hooks are configured despite `skipAutoPermissionPrompt: true` (full auto mode). Hooks could enforce quality gates automatically:
- **Pre-commit hook:** Run affected unit tests before allowing commit
- **Post-edit hook:** Trigger lint/build check after file edits
- **Agent completion hook:** Validate developer agent output against architectural rules before accepting

This would elevate D4 (Self-verification) from score 2 → 3 and make the pipeline more robust.

### 3. Close the Feedback Loop (Reviewer → Developer)

Currently the pipeline is linear: developer produces code, reviewer audits it, human decides what to fix. Wire the reviewer findings back to the developer agent automatically — if reviewer issues FAIL, spawn a developer agent with the specific findings to fix. This creates a self-correcting pipeline without human intervention.

---

## 9. Methodology & Data Window

- **All 285 sessions** counted deterministically via aggregation script (Node.js, read-only pass over all JSONL transcripts + `history.jsonl`)
- **Deep-read sessions:** `5e2608c2` (844 turns, longest), `7ee65a1a` (782 turns, second longest), `fac6b23e` + subagents (330 sub turns, orchestration), `d22e346e` + subagents (PR review via MCP), `25006621` + subagents (CSP testing), `653b536a` / `7a572f5b` / `91c6a910` (3 most recent)
- **Ceiling rule:** Stage = highest stage whose Set of AI Actions the developer routinely demonstrates. Walk top-down from Stage 4.
- **Practice-not-possession thresholds:** ≥3 invocations, across ≥2 distinct sessions, last used ≤60 days. Only PRACTICED capabilities count as stage evidence.
- **Caveat:** Analysis based on local `~/.claude` data only. Sessions conducted on other machines, in other IDEs, or before the data window are not captured. Command renames (e.g., `/business-analyst` → `/nbs-analyst`) may understate individual capability practice counts while the combined evidence across aliases is strong.
