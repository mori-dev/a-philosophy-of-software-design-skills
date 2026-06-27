---
name: aposd-refactor-plan
description: Plan a refactoring through APoSD lens. No code changes; strategy only. Breaks into PR-sized scopes.
usage: "/aposd-refactor-plan [module or feature name]"
aliases:
  - /aposd-plan
---

# aposd-refactor-plan

Plan a refactoring without writing code. Identifies the root complexity problem, proposes a safe, incremental path forward in PR-sized chunks.

## Purpose

This command is for situations where:

- "This module is hard to change and we want to fix it"
- "Every feature we add requires touching 5 files; how do we fix that?"
- "We want to refactor, but where do we start?"

It does NOT write code or propose major architectural changes. It gives you a step-by-step plan.

## Usage

```bash
# Plan refactoring for a module
/aposd-refactor-plan src/services/user_service.py

# Plan refactoring for a feature area
/aposd-refactor-plan "user authentication flow"

# Plan refactoring with additional context
/aposd-refactor-plan "error handling" --context "too many exception types scattered"
```

## Output Format

```
# APoSD Refactoring Plan: [Module/Feature]

## Root Cause Analysis

### Primary Red Flag: [Name]
- Evidence: [What symptom points to this]
- Impact: [How this harms maintainability]
- Why it happened: [Design decision or lack thereof]

### Secondary Red Flags
[Other related red flags]

## Current State

### What Works Well
[Aspects to preserve]

### What's Broken
[Specific friction points; e.g., PR to add feature X required 5-file changes]

## Proposed Path

### Phase 1: [Scope Name]
**Goal**: [Specific, measurable change]
**Changes**:
- [Concrete change 1]
- [Concrete change 2]
**Test Impact**: [Which tests update; why]
**Risk**: [Will this break existing callers? No/Low/Medium with mitigation]
**Before/After**: [Code example showing improvement]

### Phase 2: [Next Scope]
[Same structure]

[Additional phases as needed]

## Alternatives Considered

### Alternative A: [Different approach]
- Why not: [Reason for preference]

### Alternative B: [Another approach]
- Why not: [Reason for preference]

## Safety Checklist
- Does plan deepen existing modules, not create new ones?
- Does plan avoid introducing new layers/abstractions without justification?
- Is each phase one PR-sized chunk (reviewable in <30 min)?
- Can phases be merged in any order, or are there dependencies?
- Does plan reduce change amplification?
- Will callers' code become simpler?

## Merge Order & Dependencies
[If phases depend on each other, describe; if independent, say so]

## Success Metrics
[How will you know this refactor helped?]

## Estimated Effort
[Overall scope: small / medium / large; reasoning]
```

## When to Use

- **Planning a refactor before coding** — align on the path first
- **Presenting a design to a team** — show the step-by-step plan
- **Breaking large refactors into manageable PRs** — ensures each PR is reviewable
- **Understanding why a module is hard to change** — see the root cause

## When NOT to Use

- **Already have a refactor in progress** — use `/aposd-pr-review` to check the current changes
- **Making a quick fix** — refactoring plans are for longer efforts
- **Scope already decided** — if the team decided to "split into microservices", this might surface concerns

## Example: Planning an Information Leakage Fix

**Input**:
```bash
/aposd-refactor-plan src/api/handlers.py
```

**Output** (abbreviated):
```
# APoSD Refactoring Plan: User API Handler

## Root Cause Analysis

### Primary Red Flag: Information Leakage
- Evidence: Handler imports internal User model and checks user.status directly
- Impact: Status enum changes require updates in handler and model
- Why: Handler directly uses domain model instead of transformation layer

## Current State
### What's Broken
- PR to change User.status touches handler, service, model, and tests
- Handler couples API response to internal User structure

## Proposed Path

### Phase 1: Create UserResponse Type
**Goal**: API returns UserResponse, not raw User
**Changes**:
- Add UserResponse class (id, name, email only)
- Update handler to return UserResponse
- Add from_user() converter
**Test Impact**: Handler tests return UserResponse instead of User
**Risk**: Low; internal change only
**Before/After**: [Code example]

### Phase 2: Move Status Check to Service
**Goal**: Handler doesn't know about status enum
**Changes**:
- Add is_verified() method to UserService
- Handler calls service.is_verified() instead of checking status
- Remove UserStatus import from handler
**Test Impact**: Service tests update; handler tests simplified
**Risk**: Low; refactor, no new behavior
**Before/After**: [Code example]

## Success Metrics
- PR to change User.status only touches one file (model)
- Handler tests don't import UserStatus
- Handler code is 20% shorter (no status checking)

## Estimated Effort
Medium (2-3 small PRs, 2-3 hours total)
```

## Output Language

Like `/aposd-pr-review`, responds in your repository's language or prompt language.

- If repo has `.claude/settings.json` or README in Japanese, responds in Japanese
- Otherwise, responds in English

Override with `--lang ja` or `--lang en`.

## Related Commands

- `/aposd-pr-review` — Review a PR diff (this command is for planning)
- `/code-review` — General code quality (not design-specific)

## Related Skills

- `aposd-core` — Core APoSD principles and vocabulary
- `skills/aposd-core/rules/safety-rules.md` — Constraints applied to plans
- `skills/aposd-core/profiles/python.md` — Python-specific patterns
- `skills/aposd-core/profiles/typescript.md` — TypeScript-specific patterns

## Design Principles This Command Enforces

1. **Deepen existing modules over creating new ones**
2. **Avoid broad rewrites; split into PR-sized chunks**
3. **Don't recommend DDD/Clean Architecture without root cause**
4. **Measure the actual problem before proposing the solution**
5. **Make each PR independently reviewable and safe to merge**
6. **Preserve what works while fixing what's broken**
