---
name: aposd-pr-review
description: Review a PR diff through the lens of APoSD red flags. No code changes; assessment only.
usage: "/aposd-pr-review"
aliases: []
---

# aposd-pr-review

Review the current diff (staged and unstaged changes) through the lens of A Philosophy of Software Design.

## Purpose

This command detects APoSD red flags in code and ranks them by severity and effort. It does NOT propose code changes, only assessment and next steps.

## What It Does

1. **Analyzes the diff** — Reads current git changes
2. **Detects red flags** — Maps patterns to APoSD violations (shallow module, information leakage, etc.)
3. **Ranks issues** — Separates blocking (fix in this PR) from non-blocking (address later)
4. **Avoids broad rewrites** — Refuses to recommend large architectural changes in one PR
5. **Respects constraints** — Does not recommend DDD, Clean Architecture, or new layers without root cause

## Usage

```bash
/aposd-pr-review
```

Reads `git diff HEAD` and produces a review.

## Output Format

```
# APoSD Code Review

## Summary
[High-level description of changes and design impact]

## Complexity Assessment

### Blocking Issues
[Issues that should hold this PR]

**[Red Flag Name]** — `file.py:line_range`
- Problem: [Specific evidence]
- Risk: [Why this matters]
- Suggestion: [Small, reviewable fix]

### Non-Blocking Observations
[Issues to note but don't block this PR]

**[Red Flag Name]** — `file.py:line_range`
- Pattern: [What you see]
- Follow-up: [Suggested refactoring; can be separate PR]

### Safety Check
- Does this PR introduce broad rewrites? [Yes/No]
- Does this PR defer hard design decisions? [Yes/No]
- Does this PR recommend DDD/Clean Architecture without justification? [Yes/No]
- Does this PR create new abstractions without deepening existing ones? [Yes/No]

## Recommendation
[Approve / Request Changes / Approve with Follow-Up]
```

## When to Use

- **Before merging a design-critical PR** — especially those touching repositories, services, error handling, API boundaries
- **To understand the complexity impact of changes** — not just code quality, but maintainability
- **To check for red flags you might have missed** — provides a second opinion

## When NOT to Use

- **High-urgency bug fix** — use this on the follow-up refactor, not the fix itself
- **Scope already decided** — if architectural direction is mandated, this skill might surface concerns but won't override decisions

## Safety Rules Applied

This command enforces the rules in `skills/aposd-core/rules/safety-rules.md`:

- Don't split a module just because it's long
- Don't introduce layers (DDD/repository/service) as default
- Avoid adding wrapper classes just to "group" code
- Don't defer hard decisions via configuration
- Don't use messaging/events to hide coupling
- Don't proliferate abstraction levels
- Measure what's broken before refactoring
- When in doubt, deepen existing modules
- Avoid scattering error handling

## Output Language

The command detects your repository's language (or prompt language) and responds in that language.

- If the repo has a `.claude/settings.json` or README in Japanese, responds in Japanese
- Otherwise, responds in English

Override by prefixing: `/aposd-pr-review --lang ja` or `/aposd-pr-review --lang en`

## Related Commands

- `/aposd-refactor-plan` — Plan a refactor without changing code; suitable for follow-up PRs
- `/code-review` — General code quality review (not design-specific)

## Related Skills

- `aposd-core` — Core APoSD principles, red flags, and vocabulary
- `skills/aposd-core/rules/red-flags.yaml` — Red flag reference
- `skills/aposd-core/profiles/python.md` — Python-specific patterns
- `skills/aposd-core/profiles/typescript.md` — TypeScript-specific patterns
