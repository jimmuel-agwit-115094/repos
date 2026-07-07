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
| [.claude/rules/frontend/frontend.md](.claude/rules/frontend/frontend.md) | Angular patterns — components, services, HTTP, forms, routing, shared module, NBS library usage (CSP) |
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

Specialized subagents in `.claude/agents/`. Invoked by `/craft` and `/review-pr` commands via the Agent tool.

| Agent | File | Role |
|-------|------|------|
| `story-analyst-agent` | [.claude/agents/story-analyst-agent.md](.claude/agents/story-analyst-agent.md) | Fetches ADO story, produces structured business brief |
| `architect-agent` | [.claude/agents/architect-agent.md](.claude/agents/architect-agent.md) | Explores codebase, designs implementation plan |
| `developer-agent` | [.claude/agents/developer-agent.md](.claude/agents/developer-agent.md) | Writes production code + tests, self-audits |
| `qa-agent` | [.claude/agents/qa-agent.md](.claude/agents/qa-agent.md) | Audits implementation, issues SHIP / BLOCK verdict |
| `draft-pr-agent` | [.claude/agents/draft-pr-agent.md](.claude/agents/draft-pr-agent.md) | Creates DRAFT PR on ADO with exact story title |
| `pr-reviewer-agent` | [.claude/agents/pr-reviewer-agent.md](.claude/agents/pr-reviewer-agent.md) | Reviews PRs against NBS standards, posts comments |

## Micro-Skills (Agent Skills)

Focused knowledge units preloaded into agents. Lives in `.claude/skills/{name}/SKILL.md`. Not invoked directly — agents carry them.

| Skill | What it teaches |
|-------|----------------|
| `ado-story-reader` | Fetch ADO work items via MCP; extract title, AC, linked items |
| `nbs-conventions` | Which convention docs to load; key C#, Angular, CSP rules |
| `codebase-explorer` | Systematic search strategy: keywords → Glob/Grep → reference patterns |
| `story-analyzer` | Map AC to domain, surface risks, produce structured brief |
| `implementation-designer` | Design layered implementation; naming; reference pattern discipline |
| `code-implementer` | C# + Angular rules: layers, auth, feature flags, DTOs |
| `test-implementer` | Unit, integration, SPA, E2E test rules |
| `implementation-auditor` | Full audit checklist: naming, tests, security, layer discipline |
| `draft-pr-creator` | Create DRAFT PR on ADO/GitHub with exact story title |
| `pr-diff-reader` | Fetch PR metadata, full diff, and file content |
| `pr-commenter` | Format and post PR threads via ADO MCP |

## Commands

Two commands. No intermediate steps — `/craft` is fully autonomous.

| Command | Invoke | Description |
|---------|--------|-------------|
| [craft](.claude/commands/craft.md) | `/craft <id>` | Full autonomous pipeline: analyze → architect → develop → QA → Draft PR |
| [review-pr](.claude/commands/review-pr.md) | `/review-pr <pr-link>` | Review ADO PR, present findings, post approved comments |

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
