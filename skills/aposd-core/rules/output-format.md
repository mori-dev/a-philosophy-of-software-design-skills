# Output Format: APoSD Review Commands

This document defines the structure and content of outputs from `/aposd-pr-review` and `/aposd-refactor-plan`.

## `/aposd-pr-review` Output Format

**Purpose**: Review a PR diff through the lens of APoSD red flags and safety rules. No code changes; assessment only.

**Input**: Current git diff (staged and unstaged changes)

**Output Structure**:

```
# APoSD Code Review

## Summary
[1-3 bullet points: what the PR changes, at high level]

## Complexity Assessment
[Red flags detected, ranked by severity and effort]

### Blocking Issues
[Issues that should delay or block this PR]

**Issue 1: [Red Flag Name]**
- File: `path/to/file.py:line_range`
- Problem: [Brief description of the red flag]
- Risk: [Why this matters now; what breaks if we ignore it]
- Suggestion: [Small, reviewable fix for this PR, or defer to follow-up]

[Repeat for each blocking issue]

### Non-Blocking Observations
[Issues worth noting but don't block this PR]

**Observation 1: [Red Flag Name]**
- File: `path/to/file.py:line_range`
- Pattern: [What you see]
- Follow-up: [This should be addressed in a separate PR; link to refactor plan if exists]

[Repeat for each observation]

### Safety Check
- [ ] Does this PR introduce broad rewrites? (multiple layers touched, 10+ files changed)
- [ ] Does this PR defer hard design decisions via new config/flags?
- [ ] Does this PR recommend DDD/Clean Architecture without root cause?
- [ ] Does this PR create new abstractions without deepening existing modules?

[Pass/fail for each]

## Recommendation
[Approve / Request Changes / Approve with Follow-Up]

---

### Recommendation: Approve
- The PR improves or maintains design quality
- No red flags that require this PR to address
- Design decisions are sound

### Recommendation: Request Changes
- Red flags must be fixed in this PR
- Risk is too high to defer
- The fix is small enough for this PR

### Recommendation: Approve with Follow-Up
- PR is sound; approve as-is
- Some red flags noted but don't block this PR
- Suggest a follow-up PR or refactoring plan
```

## `/aposd-refactor-plan` Output Format

**Purpose**: Plan a refactoring WITHOUT changing code. Identify the root problem, propose a safe, incremental path forward.

**Input**: A module or feature area to refactor

**Output Structure**:

```
# APoSD Refactoring Plan

## Target
- File(s): [List of files affected]
- Scope: [What aspect is the problem?]

## Root Cause Analysis

### Primary Red Flag: [Name]
- Evidence: [What symptom(s) point to this red flag]
- Impact: [How does this harm maintainability]
- Why it happened: [Design decision or lack thereof that led here]

### Secondary Red Flags
[If multiple red flags, list them and their relationships]

## Current State

### What Works Well
[Aspects of the current design to preserve]

### What's Broken
[Specific friction points; e.g., "PR to add feature X required changes in 5 files"]

## Proposed Path

### Phase 1: [Scope Name] (Estimated PR count: 1-2)
**Goal**: [Specific, measurable change]
**Changes**:
- Move [logic] from `file_a` to `file_b` (reason: information hiding)
- Rename [method] to clarify intent
- Extract [special case] into separate method
**Test Impact**: [Which tests must update; why]
**Risk**: [Will this break existing callers? No/Low/Medium with mitigation]
**Before/After**: [Quick code example showing the improvement]

### Phase 2: [Next scope] (Estimated PR count: 1-2)
[Same structure]

[Repeat for each logical phase]

## Alternatives Considered

### Alternative A: [Different approach]
- Why not: [Why this isn't preferred]

### Alternative B: [Another approach]
- Why not: [Why this isn't preferred]

## Safety Checklist
- [ ] Plan deepens existing modules (not create new ones)
- [ ] No new layers or abstractions added without justification
- [ ] Each phase is one PR-sized chunk
- [ ] Phases don't require merging in a specific order (independent)
- [ ] Plan reduces change amplification, not increases it
- [ ] Callers' code becomes simpler, not more complex
- [ ] No config/flags added to defer hard decisions

## Merge Order & Dependencies
[If phases have dependencies, describe order; if independent, say so]

## Success Metrics
[How will you know this refactor helped? e.g., "new features add 1 file per feature, not 3+"]

## Estimated Effort
[Overall scope: small / medium / large; reasoning]

---

## Example: Red Flag Template for Comments

When citing red flags in either output format:

```
**[Red Flag Name]** — [file:line or file range]
- Symptom: [Observable evidence]
- Impact: [Why this matters]
- Fix: [Proposed change, or defer to follow-up]
```

Example:
```
**Information Leakage** — `service.py:45-60`
- Symptom: Caller imports `models.User` and checks `user.status == UserStatus.PENDING`
- Impact: Change to User status enum ripples to all callers
- Fix: Define a check method in the repository: `user.is_pending()` 
  Then callers use `is_pending()` and don't need the enum
```

---

## Language Specifics

### For PR Review Comments

**In the PR**: Use appropriate template from `templates/` directory.

- `templates/pr-comments-en.md` for English-speaking teams
- `templates/pr-comments-ja.md` for Japanese-speaking teams

Each template includes:
- Blocking issue template
- Observation template
- Suggestion template
- Question template

Follow the template structure exactly so PR authors understand the severity and scope.

### For Refactoring Plans

Same format regardless of language; proposal is technical and language-agnostic.

---

## Tone & Phrasing

### DO
- Be specific: "This information leakage causes a 5-file change when User schema updates"
- Be actionable: "Move the status check into the repository"
- Assume good intent: "This pattern is common, but here's a better way"
- Respect constraints: "This isn't blocking this PR, but worth a follow-up"

### DON'T
- Use vague language: "This is bad design"
- Demand perfection: "This needs a complete rewrite"
- Ignore context: "Never use X" (without root cause)
- Overwhelm: 20 separate issues in one review
- Be prescriptive without reason: "Refactor this to use a strategy pattern"

---

## When to Use "Blocking" vs "Non-Blocking"

### Blocking (must fix in this PR)
- Introduces new red flags not in the original code
- High risk with low effort (easy fix, big impact)
- Violates safety rules (broad rewrite, new layers, deferred decisions)
- Existing tests would fail or be misleading

### Non-Blocking (address later)
- Pre-existing red flags not made worse
- Would require broad changes (>1 PR)
- Requires architectural decisions that should be separate
- Low friction; refactoring can happen on its own timeline

### Follow-Up (optional, but recommended)
- Suggest a refactoring plan or a separate PR
- Include context: "Here's why this matters, and here's a small starting point"
- Don't make it urgent; let the team prioritize
