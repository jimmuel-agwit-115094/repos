---
name: story-getter
description: Retrieve and normalize an Azure DevOps work item.
argument-hint: <work-item-id>
type: agent-skill
---

Retrieve Azure DevOps work item `$ARGUMENTS` using the configured ADO MCP tools.

Collect only information relevant to understanding the story:

- ID, type, and title
- description
- acceptance criteria
- requirements and constraints
- relevant context from linked work items or discussions

Inspect linked items only when needed to understand the story.

Return:

# Story

## Work Item
- ID:
- Type:
- Title:

## Description

## Acceptance Criteria

## Requirements and Constraints

## Relevant Context

## Requirement Gaps

Preserve the original meaning and specific business rules.

Do not inspect the repository.
Do not design or propose an implementation.
Do not invent or assume missing requirements.

Keep the result concise and return only the normalized story context.
