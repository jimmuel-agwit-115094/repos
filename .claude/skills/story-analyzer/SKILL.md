---
name: story-analyzer
description: How to deeply analyze an ADO story and extract business intent, AC, domain, risks, and constraints
type: agent-skill
---

# Story Analyzer

Deep story comprehension. Goal: fully understand business intent so the architect can design with confidence. No implementation details — no file names, no class names, no code at this stage.

## What to Analyze

Work through every question below. Read every word of the story.

### Business Intent
- What problem does this solve? Who is affected and how?
- What does the user or system gain when this is done?
- Is this a new capability, a change to existing behavior, or a bug fix?

### Acceptance Criteria
- List every AC item verbatim.
- For each: interpret what "done" looks like in concrete terms — what state change must occur, what must the user see or experience.
- Flag any AC item that is vague or contradictory.

### Domain and Entities
- What business entities are involved? (e.g. Payment, Order, Tenant, Schedule, Merchant)
- What state transitions or data changes are implied?
- Are there named fields, statuses, or codes that must be preserved exactly?

### Business Rules and Constraints
- Explicit rules stated in the story (e.g. "only for tenants with X enabled", "must not affect Y", "limited to role Z").
- Implied constraints from the domain (e.g. PCI scope, idempotency, tenant isolation).

### Cross-Service Signals
- Does this mention data that lives in another service?
- Does it mention notifications, events, or side effects that span services?
- Does it mention anything a client or external consumer would see?

### Risks and Complexity
- Any indication of data migration, schema change, or backward-incompatible change?
- Any mention of performance sensitivity, concurrency, or high-volume data?
- Any dependency on another story, team, or external system?

## Output Structure

Return a structured brief — plain English, no code, no file paths:

```
STORY BRIEF
===========
Story: {id}
Title: {exact verbatim title}

What This Is Asking For:
{2–4 sentences. Plain English. Problem → who is affected → what the system gains.}

Acceptance Criteria:
| # | Criterion (verbatim) | What "done" looks like |
|---|---------------------|----------------------|
| 1 | {verbatim} | {concrete observable outcome} |

Domain Keywords:
{Terms to search for in the codebase: entity names, status values, named fields, event names}
- {keyword}

Affected Domain Area:
Primary: {service group or specific service}
Secondary: {other areas if any, or "None"}

Business Rules and Constraints:
- {rule}

Data Entities:
| Entity | Operation | Notes |
|--------|-----------|-------|
| {name} | Create/Read/Update/Delete | {state changes, new fields} |

Cross-Service Signals:
- {signal or "None apparent"}

Open Questions:
1. {ambiguity requiring decision}

Risk Signals:
- {risk or "None identified"}
```
