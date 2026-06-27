---
name: aposd-core
description: APoSD-driven code review for complexity reduction. Detects red flags, plans safe refactors, avoids broad rewrites.
tags: design-review, refactoring, complexity, software-design
---

# APoSD Core: Complexity-Driven Code Review

Operationalizes "A Philosophy of Software Design" for Claude Code to detect complexity sources, assess their impact, and plan safe incremental improvements.

## Purpose

This skill is NOT a tutorial or summary of APoSD. Instead, it gives Claude Code:

1. **Red flag detection** — patterns that indicate shallow modules, information leakage, vague naming, etc.
2. **Priority assessment** — which problems matter most for maintainability
3. **Safe refactoring** — small, reviewable changes that deepen existing modules instead of proliferating abstractions
4. **Language-specific guidance** — Python, TypeScript red flags and patterns

## Core Vocabulary

### Deep Module vs Shallow Module

- **Deep module**: simple interface hiding powerful, complex implementation (good)
- **Shallow module**: complex interface relative to the implementation it provides (bad)
- Test: if the interface documentation is longer than the implementation, it's shallow

### Information Hiding & Locality

- **Information hiding**: internal decisions shouldn't leak into the interface
- **Locality**: related operations should be in the same module; callers shouldn't orchestrate internal logic

### Change Amplification

- Change in one place cascades to many others
- Symptom: a bug fix touches 5+ files
- Root causes: scattered logic, weak module boundaries, too-generic abstractions

### Cognitive Load

- How much must you hold in mind to understand one module
- Symptom: function signature with 7+ boolean parameters, mental mapping of state machine spread across files

### Red Flags

This skill watches for these patterns:

- **Shallow module** — interface complex relative to implementation
- **Information leakage** — callers know too much about internal structure (imports internals, references storage details)
- **Change amplification** — one logical change scattered across modules
- **Vague naming** — `data`, `process`, `handle*`, `convert*` (what does it do?)
- **Special-case mixture** — general logic mixed with special cases for specific callers
- **Excessive configuration** — too many flags/options replacing hard decisions
- **Comments that repeat code** — `x += 1  # increment x` instead of addressing non-obviousness
- **Unnecessary error-path complexity** — try-catch blocks, exception handling scattered instead of defined out of existence
- **Pass-through abstractions** — layer that does nothing but forward calls (repository ↔ service ↔ controller)

## What This Skill Does NOT Do

- Recommend DDD, Clean Architecture, or layered architecture as defaults
- Push for splitting files/classes just because they're long
- Create wrapper classes or new abstractions to "organize" code
- Suggest moving to message queues, events, or polymorphism without root-cause evidence
- Treat every change as a refactoring opportunity

## What This Skill DOES Do

- Identify which complexity is harming maintainability right now
- Suggest deepening existing modules (more powerful, simpler interface)
- Propose small, safe changes that don't break existing callers
- Flag if a "fix" would trigger broad rewrites
- Distinguish "block this PR" from "follow up later"

## Usage

### Code Review (`/aposd-pr-review`)

```
/aposd-pr-review
```

Reads the current diff and:
1. Detects red flags
2. Ranks by severity and reviewer time cost
3. Suggests small changes or flag for follow-up
4. Avoids recommending large architectural changes in this PR

Output: `Blocking` (must fix), `Non-blocking` (review considers), `Follow-up` (future PR)

### Refactor Planning (`/aposd-refactor-plan`)

```
/aposd-refactor-plan
```

Without changing code, reads a module and:
1. Identifies which red flag is causing the most friction
2. Outlines a safe refactor path
3. Breaks it into 1-PR-sized scopes
4. Assesses merge risk (does this touch 10+ files?)

Output: step-by-step plan, not code changes

## Safety Rules

### Broad Rewrite Prevention

1. **Don't split without root cause** — "this file is 500 lines" alone isn't a reason
2. **Don't layer without root cause** — DDD/repository/service patterns only if information hiding demands it
3. **Don't wrapper-ize** — adding a class because two methods are related doesn't reduce complexity
4. **Don't configure instead of deciding** — flags/toggles that defer hard design choices
5. **Don't merge special cases into general code** — first, ask if the special case should exist at all

### Incremental Deepening

1. Start by understanding the current module's responsibility
2. Ask: does the interface reveal too much? Move those details inward
3. Ask: is this responsibility scattered? Gather it into one place
4. Ask: could callers be simpler if this module were more powerful? Make it so

### Modules to Deepen First

- Utility/helper layers (repositories, mappers, adapters)
- Error handling code (catch blocks, error types scattered)
- Configuration and startup logic
- API request/response serialization

Then move to business logic if patterns remain.

## When to Use

- **Code review** of a PR that touches design-critical paths (repositories, services, error handling, API boundaries)
- **Refactoring planning** when you feel the codebase is "getting messy" but can't articulate why
- **Design review** for new modules (is the interface obvious? does it hide enough?)
- **Team standard** for evaluating when to split a module vs deepen it

## When NOT to Use

- **High-urgency bug fix** — use this on the follow-up refactor, not on the fix PR
- **Performance optimization** — use a profiler first; this skill focuses on maintainability
- **Scope already decided** (e.g., "we must migrate to microservices") — this skill questions that premise

## Related Documents

- `rules/red-flags.yaml` — red flag reference, symptoms, fixes
- `rules/safety-rules.md` — broad rewrite prevention checklist
- `rules/output-format.md` — command output structure
- `templates/pr-comments-ja.md` — Japanese PR comment templates
- `templates/pr-comments-en.md` — English PR comment templates
- `profiles/python.md` — Python-specific red flags
- `profiles/typescript.md` — TypeScript-specific red flags

## Key Principle

**Prefer deepening existing modules over creating new abstractions.**

This is the core discipline that keeps complexity manageable. The skill enforces it.
