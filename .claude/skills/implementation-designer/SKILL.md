---
name: implementation-designer
description: How to design a layered NBS implementation — simplest approach, naming, reference patterns
type: agent-skill
---

# Implementation Designer

Design the simplest, cleanest, most maintainable implementation grounded in the actual codebase. You never invent patterns — you find the closest existing implementation and mirror it.

## Principles

- **Simplest** — if one layer solves it, do not add a second
- **Cleanest** — every name is obvious, every method does one thing
- **Most readable** — a new developer understands it without asking anyone
- **Most maintainable** — follows existing patterns so future changes are predictable

Before every design decision ask:
- Is this layer actually required, or am I adding it out of habit?
- Is there an existing method I can call instead of creating a new one?
- Does the naming make the intent obvious without reading the body?
- Does this follow the reference pattern exactly, or am I diverging without reason?

## Implementation Plan Structure

Produce a plan with these sections, grounded in actual file paths and real patterns:

### Summary
One paragraph. Plain English. What is being built technically, which layers it touches, and why this approach is right. Written for a developer who just picked up the ticket.

### Acceptance Criteria Coverage
| # | Criterion | Implemented by |
|---|-----------|---------------|
| 1 | {verbatim AC} | Step N — {brief description} |

### Affected Repos and Layers
| Repo | Layer | Change Type | Key Files |
|------|-------|-------------|-----------|
| {Repo} | {Layer} | Add / Modify / Create | `src/{Layer}/{File}.cs` |

### Reference Pattern
The closest existing implementation. State: file path, line range, why it's similar.

### Implementation Steps (ordered)
For each step:
- **What:** plain-English description
- **Files to create/modify:** exact paths + what changes
- **Pattern to follow:** exact file path + line
- **Code sketch:** method signature + key logic (not full code)
- **Conventions check:** naming, layer direction, auth attribute, no `Version=`
- **Existing tests affected:** file + test method + why it breaks + what to fix

### Contracts and Events
List every new DTO, event class, or client contract change.

### Database Changes
Collection name, new fields, new indexes, migration script needed (yes/no).

### Feature Flag
Required? Flag key, default, rollout plan.

### Test Plan

**Tests to update** (existing tests that will break):
| Type | File | Test method | Why it breaks | What to change |

**Tests to write** (new coverage):
- 1 happy path per AC item
- 1–2 edge cases per AC item — only realistic failure modes
- Style must match existing tests exactly

| Type | File | Test method name | Scenario | Edge case? |

### Open Questions
Any unresolved ambiguities from the story brief, or new questions from codebase exploration.

### Risks and Notes
DTO changes that affect consumers, high-volume collection indexes, coordination required.

## Naming Rules

Follow these exactly — do not invent:
- Classes: `{Feature}{Layer}` (e.g. `PaymentService`, `OrderAccessor`)
- Methods: verb-first, camelCase (e.g. `GetPaymentByIdAsync`, `CreateOrderAsync`)
- DTOs: `{Action}{Entity}Request` / `{Action}{Entity}Response`
- Events: past-tense `{Entity}{Action}Event` (e.g. `PaymentProcessedEvent`)
- Test methods: `MethodName_Condition_ExpectedResult`
