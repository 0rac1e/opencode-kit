---
name: coder-rules
description: Implementation rules and patterns for coder command. Covers: 5 CRITICAL rules (plan-only, import matrix, clean domain, no log+return, tests pass), evaluate protocol (PROCEED/REVISE/RETURN), dependency-ordered implementation.
---

# Coder Rules

## 5 CRITICAL Rules

- RULE_1 Plan Only: Implement ONLY what's in the plan. No improvements.
- RULE_2 Import Matrix: NEVER violate the import matrix.
- RULE_3 Clean Domain: NEVER add encoding/json tags to domain entities (tags belong in DTOs).
- RULE_4 No Log+Return: NEVER log AND return error simultaneously.
- RULE_5 Tests Pass: Code NOT ready until tests pass.

## Evaluate Protocol

Before implementation, critically evaluate plan (Phase 1.5):
- PROCEED: Plan is implementable as-is → start implementation
- REVISE: Minor gaps, can fix inline → note adjustments, proceed
- RETURN: Major gaps or feasibility issues → return to /plan-review with feedback

Evaluate checks: feasibility, hidden complexities, edge cases, performance, dependencies.
Output: `.opencode/prompts/{feature}-evaluate.md`

Evaluate has an exploration budget (SEE coder.md → evaluate_budget).
When budget is reached, DECIDE with available information.
Prefer PROCEED with notes over endless research.
The planner already researched — evaluate is VALIDATION, not discovery.

## Spec Check Protocol

After VERIFY passes, run spec compliance self-check (Phase 3.5):
- PASS: all Parts covered, scope respected, AC traceable → proceed to handoff
- PARTIAL: minor gaps documented → proceed with gaps noted in handoff
- FAIL: missing Part → inline fix (max 1 retry) → re-run VERIFY → re-check

## Instructions

### Step 1: Load plan and verify approval
Read `.opencode/prompts/{feature}.md`. Verify plan passed plan-review.
If plan not found → ERROR, exit. If not approved → ERROR, exit.

### Step 2: Run Evaluate Protocol
Before writing ANY code, evaluate the plan critically.
Decision: PROCEED / REVISE / RETURN (see Evaluate Protocol above).
Write output to `.opencode/prompts/{feature}-evaluate.md`.

### Step 3: Implement parts in dependency order
Follow lower-layers-first: data access → models → domain → API → tests → wiring.
After each Part: PostToolUse hooks auto-format files (gofmt). Run LINT only for import/error checks. Check 5 CRITICAL Rules above continuously.
Do NOT run tests (make test, go test) between Parts. Tests run ONCE at Step 4 VERIFY.

### Step 4: Verify
Run full VERIFY: `go vet ./... && make fmt && make lint && make test`.
If tests fail 3x → load systematic-debugging skill, run Phase 1 root cause investigation.
On success → proceed to Step 5.

### Step 5: Spec Check and form handoff
Run SPEC CHECK. S complexity: lightweight (coverage only).
If FAIL: inline fix → re-run VERIFY → re-check (max 1 retry).
On PASS/PARTIAL → form handoff payload for code-review, including spec_check.

## Example

### Clean Domain — no json tags in entities (RULE_3)

**Good:**
```go
type Service struct {
    ID string
}
```

**Bad:**
```go
type Service struct {
    ID string `json:"id"`
}
```
**Why:** RULE_3 — Domain entities must be pure. No encoding/json tags. Tags belong in DTOs at the handler/API layer.

## Common Issues

### Tests fail 3x in a row — stuck
**Cause:** Bug in implementation logic or wrong approach.
**Fix:** Load systematic-debugging skill. Run Phase 1 (Root Cause Investigation) with
`go test -v -count=1 ./...` output as evidence. Trace data flow to find root cause.
If root cause found → implement single fix + VERIFY. If still stuck → STOP, request manual help.

### Import matrix violation detected
**Cause:** Didn't check architecture rules before implementation.
**Fix:** Review import matrix (handler → service → repository → models). Refactor imports. This is ALWAYS a BLOCKER.
