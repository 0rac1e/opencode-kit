---
name: planner-rules
description: Task analysis and planning rules for planner command. Covers: task classification (7 types), S/M/L/XL complexity routing, plan documentation with full code examples.
---

# Planner Rules

## Task Classification
7 task types:
1. new_feature — adding new functionality
2. bug_fix — fixing existing issues
3. refactor — improving code structure
4. integration — connecting external systems
5. migration — database or data changes
6. configuration — adding/changing config
7. documentation — updating docs

## Complexity Routing

| Complexity | Parts | Layers | Plan Review | Sequential Thinking | code-researcher |
|------------|-------|--------|-------------|--------------------|-----------------|
| **S** | 1 | 1 | skip | not needed | skip |
| **M** | 2–3 | 2 | standard | as needed | skip |
| **L** | 4–6 | 3+ | standard | recommended | yes |
| **XL** | 7+ | 4+ | standard | required | yes |

## Data Flow
For M/L/XL tasks, map data flow:
- Input → Processing → Storage → Output
- Layer boundaries
- Import dependencies

## Research Budget
Prevent exploration loops:
- S: 5 file reads, 12 tool calls
- M: 10 file reads, 20 tool calls
- L: 20 file reads, 35 tool calls
- XL: 30 file reads, 50 tool calls

When budget exceeded → summarize and transition to DESIGN.

## Plan Documentation
Output template:
```markdown
# Task: {Name}

## Context
[Description]

## Scope
### IN
- [ ] ...
### OUT
- ... (reason)

## Part N: {Name}
**File:** `path/file{EXT}` (CREATE/UPDATE)
[FULL code example]

## Acceptance Criteria
- [ ] LINT passes
- [ ] TEST passes
```

## Rules
- No Code: research and planning only
- Questions First: ask clarifying questions before research
- Full Examples: code examples must be FULL
- Import Matrix: check dependencies between layers
