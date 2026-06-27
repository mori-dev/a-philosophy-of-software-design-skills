# A Philosophy of Software Design — Claude Code Integration

Operationalizes "A Philosophy of Software Design" by John Ousterhout for Claude Code to detect design complexity sources, assess their impact, and plan safe incremental improvements.

## Quick Start

This repository provides two Claude Code commands:

1. **`/aposd-pr-review`** — Review a PR diff for design problems (shallow modules, information leakage, change amplification, etc.)
2. **`/aposd-refactor-plan`** — Plan a refactoring step-by-step without writing code

Both commands use the `aposd-core` skill, which contains red flag definitions, safety rules, and templates for both Python and TypeScript.

### Installation

```bash
# Copy to your Claude Code config
cp -r skills/aposd-core ~/.claude/skills/aposd-core
cp commands/aposd-*.md ~/.claude/commands/

# Test it
/aposd-pr-review
```

For detailed setup and usage, see [Setup and Usage](#setup-and-usage) below.

---

## Setup and Usage

### Installation

#### Prerequisites
- Claude Code (CLI, desktop app, or web) with command support
- Git repository with code to review

#### Option 1: Global Installation (Recommended)

```bash
# Clone or download this repository
git clone https://github.com/mori-dev/a-philosophy-of-software-design-skills.git
cd a-philosophy-of-software-design-skills

# Copy to your Claude config
cp -r skills/aposd-core ~/.claude/skills/aposd-core
cp commands/aposd-*.md ~/.claude/commands/

# Verify
ls ~/.claude/commands/aposd-*.md  # Should show 2 files
ls ~/.claude/skills/aposd-core/SKILL.md  # Should exist
```

#### Option 2: Project-Local Installation

```bash
# In your project repo
mkdir -p .claude/commands .claude/skills
cp -r <repo>/skills/aposd-core .claude/skills/
cp <repo>/commands/aposd-*.md .claude/commands/
```

After installation, use `/aposd-pr-review` and `/aposd-refactor-plan` in Claude Code.

---

### Command 1: `/aposd-pr-review`

**Purpose**: Review a PR diff through the APoSD lens. Detects red flags, ranks by severity, distinguishes blocking from non-blocking issues.

**When to use**:
- Before merging a design-critical PR
- To understand complexity impact of changes
- To check for red flags you might have missed

**Examples**:

```
/aposd-pr-review
                    # Review all current git changes

/aposd-pr-review src/services/user.py
                    # Review specific file

/aposd-pr-review "user auth flow" --lang ja
                    # With context and language override
```

**Output includes**:
- **Summary**: High-level design impact
- **Blocking Issues**: Fix in this PR or hold merge
- **Non-Blocking Observations**: Suggestions for follow-up
- **Safety Check**: Broad rewrite detection, DDD recommendations, etc.
- **Recommendation**: Approve / Request Changes / Approve with Follow-Up

**Checks for**:
- Shallow modules (simple interface, simple implementation)
- Information leakage (caller sees internal structure)
- Change amplification (1 change → 5+ files affected)
- Vague naming, special-case mixture, excessive configuration
- Error-path complexity, pass-through abstractions
- Temporal decomposition, repetition, nonobvious code

**Will NOT recommend**:
- Splitting modules just because they're long
- DDD/Clean Architecture without root cause
- Creating wrapper classes to "group" code
- Proliferating abstraction levels

---

### Command 2: `/aposd-refactor-plan`

**Purpose**: Plan a refactoring step-by-step without writing code. Breaks work into PR-sized chunks with risk assessment.

**When to use**:
- Before starting a major refactor
- When presenting a strategy to the team
- To break large refactors into reviewable PRs
- To understand why a module is hard to change

**Examples**:

```
/aposd-refactor-plan src/services/user.py
                    # Plan refactoring for a module

/aposd-refactor-plan "error handling"
                    # Plan for a concern area

/aposd-refactor-plan user_service.py --lang ja
                    # With language override
```

**Output includes**:
- **Root Cause Analysis**: Primary red flag and why it matters
- **Current State**: What works well, what's broken
- **Proposed Path**: Phase-by-phase refactoring plan
  - Each phase is 1-2 PR-sized chunks
  - Goal, concrete changes, test impact, risk assessment
  - Before/after code examples
- **Alternatives**: Why other approaches were rejected
- **Success Metrics**: How to measure if the refactor worked
- **Estimated Effort**: Small / Medium / Large

**Does NOT**:
- Write code or generate patches
- Recommend major architectural changes
- Suggest DDD/Clean Architecture defaults
- Create new layers without justification

---

## Core Philosophy

**The fundamental problem in software design is managing complexity.**

Complexity makes systems hard to understand and modify. The goal is to minimize complexity through:
- Strategic (not tactical) programming
- Deep modules with simple interfaces
- Information hiding
- Eliminating special cases and exceptions
- Good names and obvious code
- Clear abstractions

## Skills Overview

### 1. [Complexity Management](./01-complexity-management.md)
**Core Focus**: Understanding and fighting complexity

Key concepts:
- Complexity defined: change amplification, cognitive load, unknown unknowns
- Root causes: dependencies and obscurity
- Strategic vs tactical programming
- Investment mindset
- Incremental complexity accumulation

**Use when**: Starting any design, reviewing code, making design decisions

---

### 2. [Deep Modules and Abstraction](./02-deep-modules.md)
**Core Focus**: Creating powerful abstractions with simple interfaces

Key concepts:
- Deep modules: simple interface, powerful implementation
- Information hiding
- Avoiding information leakage
- Interface vs implementation
- Pass-through methods (anti-pattern)

**Use when**: Designing classes/modules, creating APIs, refactoring

---

### 3. [Error Handling](./03-error-handling.md)
**Core Focus**: Defining errors out of existence

Key concepts:
- Exceptions add complexity
- Define semantics so errors can't occur
- Mask exceptions at low levels
- Exception aggregation
- Eliminating special cases

**Use when**: Designing error handling, simplifying exception code

---

### 4. [Naming and Obviousness](./04-naming-obviousness.md)
**Core Focus**: Making code self-documenting and obvious

Key concepts:
- Names should create mental images
- Be precise, not generic
- Consistency in naming
- Making code obvious
- Avoiding surprises

**Use when**: Naming variables/methods/classes, making code clearer

---

### 5. [Comments and Documentation](./05-comments-documentation.md)
**Core Focus**: Documenting what code cannot express

Key concepts:
- Comments describe non-obvious aspects
- Write comments first (design tool)
- Interface vs implementation comments
- Different levels of detail than code
- Precision and intuition

**Use when**: Documenting code, designing interfaces, maintaining docs

---

### 6. [General-Purpose Design](./06-general-purpose-design.md)
**Core Focus**: Somewhat general-purpose is deeper

Key concepts:
- General-purpose modules are deeper
- Sweet spot: not too specific, not too general
- Different layers, different abstractions
- Avoiding pass-through methods
- Separating general and special-purpose code

**Use when**: Designing interfaces, creating reusable components

---

### 7. [Design Process](./07-design-process.md)
**Core Focus**: How to approach design

Key concepts:
- Design it twice (consider alternatives)
- Incremental design evolution
- Writing comments first
- Strategic refactoring
- Investment in design quality

**Use when**: Starting new features, refactoring, throughout development

---

### 8. [Consistency and Conventions](./08-consistency-conventions.md)
**Core Focus**: Leverage through consistency

Key concepts:
- Consistency reduces cognitive load
- Types: naming, style, interfaces, patterns
- Establishing and enforcing conventions
- Following existing patterns
- When consistency goes too far

**Use when**: Writing code, code reviews, establishing team practices

---

## Quick Reference: Red Flags

Watch for these warning signs of complexity:

- **Shallow Module**: Interface not much simpler than implementation
- **Information Leakage**: Design decision reflected in multiple modules
- **Temporal Decomposition**: Structure based on execution order
- **Overexposure**: API forces awareness of rarely-used features
- **Pass-Through Method**: Just calls another method with same signature
- **Repetition**: Same code duplicated multiple times
- **Special-General Mixture**: Special-purpose code not separated
- **Comment Repeats Code**: Comment obvious from code itself
- **Implementation Documentation Contaminates Interface**: Interface describes how, not what
- **Vague Name**: Name too generic to convey meaning
- **Hard to Pick Name**: Difficulty finding good name suggests unclear design
- **Hard to Describe**: Long documentation needed suggests complex interface
- **Nonobvious Code**: Can't understand with quick reading

## Quick Reference: Design Principles

Core principles to follow:

1. **Complexity is incremental** - Sweat the small stuff
2. **Working code isn't enough** - Design quality matters
3. **Make continual small investments** - 10-20% of time on design
4. **Modules should be deep** - Simple interface, powerful implementation
5. **Interfaces should make common usage simple** - Optimize for typical case
6. **Simple interface > simple implementation** - Hide complexity
7. **General-purpose modules are deeper** - Don't be too specific
8. **Separate general and special-purpose code** - Clear boundaries
9. **Different layers, different abstractions** - Add value per layer
10. **Pull complexity downward** - Hide in implementation
11. **Define errors out of existence** - Design so errors can't occur
12. **Design it twice** - Consider alternatives
13. **Comments describe non-obvious** - Different level than code
14. **Design for ease of reading** - Not ease of writing
15. **Increments should be abstractions** - Not features

## How to Use These Skills

### For New Projects
1. Start with 01-Complexity Management to set mindset
2. Apply 02-Deep Modules when designing architecture
3. Use 07-Design Process throughout development
4. Reference others as specific situations arise

### For Existing Projects
1. Review 01-Complexity Management for philosophy
2. Use as code review checklist (scan red flags)
3. Apply when refactoring (especially 02, 03, 06)
4. Improve naming with 04 during regular work
5. Use 08 to establish team consistency

### For Code Reviews
1. Check for red flags from Quick Reference
2. Verify principles from Quick Reference
3. Suggest specific skill for improvements
4. Focus on teaching, not just finding problems

### For Learning
1. Read skills in order (they build on each other)
2. Practice identifying red flags in code
3. Try "design it twice" on next feature
4. Experiment with writing comments first
5. Measure: is code getting simpler?

## Integration with Development

### Daily Work
- Every design decision: check against principles
- Every new method: write comment first
- Every code review: look for red flags
- Every refactoring: aim for simplicity

### Team Practices
- Share skills with team
- Use in code review discussions
- Establish conventions (skill 08)
- Hold periodic design discussions
- Measure complexity trends

### Tools
- Linters for style consistency
- Documentation generators for comments
- Complexity metrics to track trends
- Code review checklists

## Key Insights

**From the book's philosophy**:

1. **Complexity is the enemy** - Everything should aim to reduce it
2. **Strategic thinking required** - Tactical programming creates mess
3. **Invest in design** - Pays off quickly (within months)
4. **Comments are design tools** - Write them first
5. **Simple != Easy** - Simple designs take thought
6. **Abstractions are key** - Hide complexity behind clean interfaces
7. **Consistency provides leverage** - Learn once, apply everywhere
8. **Define problems away** - Best solution is no exception
9. **Obvious is crucial** - Code should be immediately understandable
10. **Incremental improvement** - Small investments accumulate

## Measuring Success

**You're succeeding when**:
- New features take less time to add
- Bugs are easier to fix
- Code reviews go faster
- New developers onboard quicker
- Less time spent understanding code
- Fewer "what does this do?" moments
- More time designing, less firefighting

**Warning signs**:
- Feature velocity decreasing
- Bug fix creates new bugs
- Can't understand code you wrote months ago
- Afraid to make changes
- Extensive documentation needed for simple tasks
- Constantly fighting the codebase

## Philosophy Summary

> "The most fundamental problem in computer science is problem decomposition: how to take a complex problem and divide it up into pieces that can be solved independently."

> "If you want to make it easier to write software, so that we can build more powerful systems more cheaply, we must find ways to make software simpler."

> "Your primary goal must be to produce a great design, which also happens to work. This is strategic programming."

> "If a software system is hard to understand and modify, then it is complicated; if it is easy to understand and modify, then it is simple."

## Additional Resources

- Book: "A Philosophy of Software Design" by John Ousterhout
- CS 190 at Stanford University (where these ideas were developed)
- Related concepts: information hiding (David Parnas), separation of concerns
- Clean Code, SOLID principles (complementary approaches)

## License Note

These skills are derived from concepts in "A Philosophy of Software Design" by John Ousterhout, published by Yaknyam Press. They represent practical applications of the book's principles for use in software development.
