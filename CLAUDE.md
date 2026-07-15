# NBS Services — Claude Code Guide

Root workspace: `C:\neldevsrc\repos`. Each subfolder is an independent repo.
Do not expand this file — link to the chunks in `.claude/rules/` below.

## Documentation Index

Docs live under `.claude/rules/` organised by concern.

**Architecture**

| File | Contents |
|------|----------|
| [.claude/rules/architecture/architecture.md](.claude/rules/architecture/architecture.md) | Service groups, solution layout, hosting, auth, feature flags |
| [.claude/rules/backend/backend.md](.claude/rules/backend/backend.md) | Backend architecture map — layers, flows, patterns, dependency rules, where to look first |

**Microservice Setup**

| File | Contents |
|------|----------|
| [.claude/rules/microservice-setup/services-payment-hub.md](.claude/rules/microservice-setup/services-payment-hub.md) | Banking, Checkout, Payments, Remittance, ScheduledPayments, PaymentMethodSelector |
| [.claude/rules/microservice-setup/services-core.md](.claude/rules/microservice-setup/services-core.md) | Encryption, EventHub, ChangeHistory, Passport, Tenant, Menu, CustomerServicePortal |
| [.claude/rules/microservice-setup/dependencies.md](.claude/rules/microservice-setup/dependencies.md) | Inter-service dependency graph, local startup order, shared packages |

**Frontend**

| File | Contents |
|------|----------|
| [.claude/rules/frontend/frontend.md](.claude/rules/frontend/frontend.md) | Angular patterns — components, services, HTTP, forms, routing, shared module, NBS library usage, NPDS web component registration (CSP) |
| [.claude/rules/frontend/frontend-libraries.md](.claude/rules/frontend/frontend-libraries.md) | xlr8AngularToolkit, xlr8PageTemplate, Framework.Web.AngularTesting |

**Conventions**

| File | Contents |
|------|----------|
| [.claude/rules/conventions/conventions.md](.claude/rules/conventions/conventions.md) | C#, Angular/TS, CI/CD, DB naming and patterns |

**Tests**

| File | Contents |
|------|----------|
| [.claude/rules/tests/backend-service-test.md](.claude/rules/tests/backend-service-test.md) | Unit tests — Services, Accessors, Client, Jobs (xUnit, Moq, FluentAssertions) |
| [.claude/rules/tests/integration-test.md](.claude/rules/tests/integration-test.md) | Integration tests — TestFixture/BaseTest pattern, seeder lifecycle, MockUserContext, bearer auth, log side-effect verification |
| [.claude/rules/tests/spa-service-test.md](.claude/rules/tests/spa-service-test.md) | SPA tests — C# controller/service/authorization unit tests + Angular Karma/Jasmine component tests |
| [.claude/rules/tests/spec-test.md](.claude/rules/tests/spec-test.md) | Angular spec patterns — ComponentElements, NbsGuardTestHelper, HTTP mocking, fakeAsync, spy setup |
| [.claude/rules/tests/e2e-test.md](.claude/rules/tests/e2e-test.md) | E2E tests — Playwright against deployed env, page objects, fixtures, CI integration |

**MCP / Tooling**

| File | Contents |
|------|----------|
| [.claude/mcp-ado.md](.claude/mcp-ado.md) | Azure DevOps MCP setup — PAT config, available tools, work item / PR queries |

## MCP Servers

| File | Server | Note |
|------|--------|------|
| [.mcp.json](.mcp.json) | `azure-devops-mcp` | Must stay at root — Claude Code only loads `.mcp.json` from project root |

## Agents

Specialized subagents in `.claude/agents/`. Invoked via commands or directly through the Agent tool.

| Agent | File | Role |
|-------|------|------|
| `nbs-analyst` | [.claude/agents/nbs-analyst.md](.claude/agents/nbs-analyst.md) | Fetches ADO story, produces estimation brief with story points, blockers, unknowns |
| `nbs-architect` | [.claude/agents/nbs-architect.md](.claude/agents/nbs-architect.md) | Explores codebase, designs full implementation plan |
| `nbs-developer` | [.claude/agents/nbs-developer.md](.claude/agents/nbs-developer.md) | Implements plan — backend, frontend, tests |
| `nbs-reviewer` | [.claude/agents/nbs-reviewer.md](.claude/agents/nbs-reviewer.md) | Audits git changes against NBS standards, issues PASS / FAIL verdict |
| `nbs-pr-reviewer` | [.claude/agents/nbs-pr-reviewer.md](.claude/agents/nbs-pr-reviewer.md) | Reviews ADO pull requests against NBS standards, posts comments |

## Micro-Skills (Agent Skills)

Focused knowledge units loaded into agents or invoked directly as slash commands. Lives in `.claude/skills/{name}/SKILL.md`.

| Skill | File | What it does |
|-------|------|--------------|
| `implementation-plan` | [.claude/skills/implementation-plan/SKILL.md](.claude/skills/implementation-plan/SKILL.md) | Template for NBS implementation plan files; writes to `.claude/implementation-plan/{story-id}.md` |
| `story-getter` | [.claude/skills/story-getter/SKILL.md](.claude/skills/story-getter/SKILL.md) | Retrieve and normalize an Azure DevOps work item |
| `test-fixer` | [.claude/skills/test-fixer/SKILL.md](.claude/skills/test-fixer/SKILL.md) | Diagnose and fix failing Angular Karma/Jasmine specs, C# WebSpa.UnitTests, and Playwright E2E tests |
| `ui-fixer` | [.claude/skills/ui-fixer/SKILL.md](.claude/skills/ui-fixer/SKILL.md) | Diagnose and fix Angular UI issues — NPDS, Material, change detection, accessibility, SCSS |

## Commands

Individual commands per pipeline stage. Each maps directly to a subagent.

| Command | Invoke | Description |
|---------|--------|-------------|
| [nbs-analyst](.claude/commands/nbs-analyst.md) | `/nbs-analyst <id>` | Fetch ADO story, produce estimation brief → writes to `.claude/for-estimation-stories/{id}.md` |
| [nbs-architect](.claude/commands/nbs-architect.md) | `/nbs-architect <id>` | Explore codebase, design full implementation plan → writes to `.claude/implementation-plan/{id}.md` |
| [nbs-developer](.claude/commands/nbs-developer.md) | `/nbs-developer <id>` | Read implementation plan, implement backend + frontend + tests |
| [nbs-reviewer](.claude/commands/nbs-reviewer.md) | `/nbs-reviewer <id>` | QA git changes against NBS standards, issue PASS / FAIL verdict |
| [nbs-pr-reviewer](.claude/commands/nbs-pr-reviewer.md) | `/nbs-pr-reviewer <pr-link>` | Review ADO PR, post inline comments and summary thread |
| [repo-update](.claude/commands/repo-update.md) | `/repo-update <repo-name>` | Register a new repo — updates workspace docs and `.gitignore` |

## Output Directories

| Directory | Written by | Contents |
|-----------|-----------|----------|
| [.claude/for-estimation-stories/](.claude/for-estimation-stories/) | `/nbs-analyst` | Estimation briefs — one `.md` file per story ID |
| [.claude/implementation-plan/](.claude/implementation-plan/) | `/nbs-architect` | Full implementation plans — one `.md` file per story ID |

## Quick Facts

- **Platform:** NBS (Nelnet Business Services) — multi-tenant payment processing
- **Backend:** .NET (C#), MongoDB Atlas, Redis, Azure Event Hubs
- **Frontend:** Angular SPAs (hosted as BFF), shared via npm libraries
- **Auth:** Passport (internal OAuth2/OpenID provider)
- **CI/CD:** Azure DevOps Pipelines → Kubernetes (Helm)
- **Feature Flags:** LaunchDarkly via `Nbs.Framework.FeatureFlags`
- **Excluded from this workspace:** `Personal Project/` (unrelated)

## Per-Service CLAUDE.md Files

Some repos have their own `.claude/` documentation:

| Repo | CLAUDE.md |
|------|-----------|
| `CustomerServicePortal` | [CustomerServicePortal/CLAUDE.md](CustomerServicePortal/CLAUDE.md) |
