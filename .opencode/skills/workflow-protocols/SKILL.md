---
name: workflow-protocols
description: Orchestration protocols for workflow pipeline. Covers: handoff contracts, checkpoint protocol, re-routing, pipeline metrics.
---

# Workflow Protocols

## Overview
Orchestration protocols for the development workflow pipeline. Load at workflow startup (step 0.1).

## Handoff Protocol
4 typed payload contracts between phases:
1. planner → plan-reviewer
2. plan-reviewer → coder
3. coder → code-reviewer
4. code-reviewer → completion

Each handoff includes:
- Artifact path
- Key decisions
- Known risks
- Areas needing attention
- Iteration count

## Checkpoint Protocol
State saved after each phase for session recovery:
- Current phase
- Phase name
- Implementation progress (parts completed/total)
- Iteration counters
- Timestamp
- Verdict (if applicable)

## Re-Routing
Pipeline adjusts route on complexity mismatch:
- Downgrade: XL → L, L → M, M → S
- Upgrade: S → M, M → L, L → XL
- Triggers: new information, scope changes, user request

## Pipeline Metrics
Track at completion:
- Total phases completed
- Iterations per review cycle
- Time spent per phase
- Complexity classification accuracy
