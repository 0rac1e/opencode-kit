---
name: code-review-rules
description: Review standards for code-reviewer agent. Covers: severity classification (BLOCKER/MAJOR/MINOR/NIT), decision matrix (APPROVED/APPROVED_WITH_COMMENTS/CHANGES_REQUESTED), auto-escalation rules, grep search patterns for automated checks.
---

# Code Review Rules

## Severity Classification

- BLOCKER: Architecture/security violation — blocks approval
- MAJOR: Error handling, logging, significant gaps — blocks approval
- MINOR: Code style, naming, documentation — does not block
- NIT: Stylistic preference — does not block

## Decision Matrix

| Verdict | BLOCKER | MAJOR | MINOR | NIT |
|---------|---------|-------|-------|-----|
| APPROVED | 0 | 0 | 0 | any |
| APPROVED_WITH_COMMENTS | 0 | 0 | any | any |
| CHANGES_REQUESTED | 1+ | 1+ | 3+ | any |

## Auto-Escalation Rules

1. 5+ MINOR in same file → escalate to MAJOR
2. Security issue (any severity) → always BLOCKER
3. Import matrix violation → always BLOCKER

## Grep Search Patterns

### Error Handling
```bash
# Find log + return patterns
grep -n "log\." internal/**/*.go | grep -A2 "return.*err"

# Find unwrapped errors
grep -n "return err" internal/**/*.go
```

### Architecture
```bash
# Find cross-layer imports
grep -n "internal/handler" internal/service/**/*.go
grep -n "internal/handler" internal/repository/**/*.go
grep -n "internal/service" internal/handler/**/*.go
```

### Security
```bash
# Find hardcoded secrets
grep -rn "password\|token\|secret\|key" --include="*.go" internal/

# Find SQL injection risks
grep -n "fmt.Sprintf.*SELECT\|fmt.Sprintf.*INSERT\|fmt.Sprintf.*UPDATE" internal/**/*.go
```

### Code Quality
```bash
# Find long functions (>30 lines)
awk '/^func/ {start=NR} /^}/ {if(NR-start>30) print FILENAME":"start":"NR-start" lines"}' internal/**/*.go

# Find TODO/FIXME/HACK
grep -rn "TODO\|FIXME\|HACK" --include="*.go" internal/
```

## Review Checklist

### Architecture
- [ ] Import matrix compliance
- [ ] Domain purity (no encoding/json tags)
- [ ] Layer boundaries respected

### Error Handling
- [ ] All errors wrapped with fmt.Errorf("context: %w", err)
- [ ] No log AND return same error
- [ ] Functions ≤ 30 lines

### Security
- [ ] No hardcoded secrets
- [ ] Parameterized SQL queries
- [ ] Input validation on handler layer

### Test Coverage
- [ ] New code has corresponding tests
- [ ] Test coverage maintained or improved
- [ ] Meaningful assertions

### Project-Specific
- [ ] Config changes documented
- [ ] Generated files not manually edited
- [ ] Mocks regenerated if interfaces changed
