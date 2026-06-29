# Output Format: APoSD Review Commands

This document defines the structure and content of outputs from `/aposd-review`,
`/aposd-pr-review`, and `/aposd-refactor-plan`.

All three reports share the enrichment defined in `rules/scoring.md`:

- **Design Health Score** (0–100) + severity count distribution + explicit scope
- **Diagnostic chain** per finding: Symptom → Root cause → Consequence → Fix, with
  the `red-flags.yaml` id, the APoSD chapter citation, and Confidence / Effort tags
- **Priority matrix** (Impact × Effort) naming the single first action
- **Mermaid dependency graph** for module-level scopes
- **Before / After** code block for every blocking finding and refactor phase

Read `rules/scoring.md` before producing any of these reports. Do not regress to a
flat bullet list of red flags.

---

## `/aposd-pr-review` Output Format

**Purpose**: Review a PR diff through the lens of APoSD red flags and safety
rules. No code changes; assessment only.

**Input**: A diff — current branch vs main, a named file, or a GitHub PR.

**Output Structure**:

```
# APoSD Code Review

## Design Health: <score> / 100 (<grade>, <preset> preset)

| Severity | 件数 | 該当 red-flag |
|----------|-----|--------------|
| 🔴 High   | N   | ...           |
| 🟡 Medium | N   | ...           |
| 🟢 Low    | N   | ...           |

スコープ: base=main, <files> ファイル変更 (+<added> / −<removed> 行)
この差分が新たに増やした赤旗: N 件 / 元から存在した赤旗: N 件

## Summary
[1–3 bullets: what the PR changes at a high level, and its net design effect]

## 着手順位（Impact × Effort）
[2×2 matrix per scoring.md §5, with the single top action named]

## Blocking Issues
[Findings that should delay or block this PR — diagnostic chain per scoring.md §3]

**Shallow Module** — `path/to/file.py:line_range`
[red-flag: shallow_module · 02-deep-modules.md · Confidence: High · Effort: S]
- Symptom: ...
- Root cause: ...
- Consequence: ...
- Fix: ...

Before / After:
[minimal code block per scoring.md §7]

[Repeat for each blocking issue]

## Non-Blocking Observations
[Worth noting but don't block this PR — same chain, Fix line says "follow-up"]

**Vague Naming** — `path/to/file.py:line_range`
[red-flag: vague_naming · 04-naming-obviousness.md · Confidence: Medium · Effort: S]
- Symptom: ...
- Root cause: ...
- Consequence: ...
- Fix: follow-up PR; link to refactor plan if one exists

[Repeat for each observation]

## Safety Check
- [ ] Broad rewrite? (multiple layers touched, 10+ files)
- [ ] Defers hard decisions via new config/flags?
- [ ] Recommends DDD/Clean Architecture without root cause?
- [ ] Creates new abstractions instead of deepening existing modules?
[Pass/fail + one-line reason for each]

## Recommendation
[Approve / Request Changes / Approve with Follow-Up]
```

### Recommendation meanings

- **Approve** — design quality maintained or improved; no red flag requires this PR to act.
- **Request Changes** — a blocking red flag must be fixed here; risk too high to defer; fix is small enough for this PR.
- **Approve with Follow-Up** — PR is sound; noted red flags don't block; suggest a follow-up PR or refactor plan.

---

## `/aposd-review` Output Format

**Purpose**: Audit a repository, directory, or file set for accumulated design
debt. Track design health over time, not a single PR.

**Input**: Repo root (no args), one or more directories, or one or more files.

**Output Structure**:

```
# APoSD Design Audit: <scope label>

## Design Health: <score> / 100 (<grade>, <preset> preset)

| Severity | 件数 | 該当 red-flag |
|----------|-----|--------------|
| 🔴 High   | N   | ...           |
| 🟡 Medium | N   | ...           |
| 🟢 Low    | N   | ...           |

スコープ: <files> ファイル / <lines> 行をスキャン（除外: ...）

## Summary
[Codebase-wide design state in 2–4 lines: where the debt concentrates]

## Module Map
[Mermaid dependency graph per scoring.md §6 — red/yellow color-coded modules.
 Skip if scope is a single file.]

## 着手順位（Impact × Effort）
[2×2 matrix per scoring.md §5, with the single top action named]

## Findings by Module
[Group findings under the module/directory they live in, worst module first.
 Each finding uses the diagnostic chain per scoring.md §3.]

### `src/services/` — 🔴 3 findings
**Shallow Module** — `user_service.py:12-40`
[red-flag: shallow_module · 02-deep-modules.md · Confidence: High · Effort: S]
- Symptom: ...
- Root cause: ...
- Consequence: ...
- Fix: ...

Before / After:
[minimal code block — blocking-equivalent findings only]

[Repeat per finding, then per module]

## Cross-Module Concerns
[Issues that span files — information leakage, change amplification, layering.
 Reference the edges drawn in the Module Map.]

## Safety Check
- 大幅な設計変更の必要性: Yes / No（理由）
- Broad rewrite の兆候: Yes / No（理由）
- DDD/Clean Architecture への移行根拠: 有 / 無（理由）

## Recommendation
[Prioritized next steps. If the worst module needs more than one PR, route it to
 /aposd-refactor-plan rather than proposing an inline fix.]
```

---

## `/aposd-refactor-plan` Output Format

**Purpose**: Plan a refactoring WITHOUT changing code. Identify the root problem,
propose a safe, incremental path.

**Input**: A module or feature area to refactor.

**Output Structure**:

```
# APoSD Refactoring Plan: <module/feature>

## Target
- File(s): [files affected]
- Scope: [what aspect is the problem]
- Design Health (current): <score> / 100 (<grade>) — projected after plan: <score>

## Root Cause Analysis

### Primary Red Flag: <name>
[red-flag: <id> · <chapter file> · Confidence: <level>]
- Symptom: [observable evidence]
- Root cause: [the design decision or omission]
- Consequence: [how it harms maintainability; which red flags it triggers]

### Secondary Red Flags
[Related flags and how they connect to the primary one]

## Current State

### Architecture (before)
[Mermaid graph per scoring.md §6 showing today's shape and the bad edges]

### What Works Well
[Aspects to preserve]

### What's Broken
[Specific friction points, e.g. "adding field X required changes in 5 files"]

## Proposed Path

### Phase 1: <scope name> (PR count: 1–2)
**Goal**: [specific, measurable change]
**Health delta**: <current> → <projected> after this phase
**Changes**:
- Move [logic] from `file_a` to `file_b` (reason: information hiding)
- Rename [method] to clarify intent
**Test Impact**: [which tests update; why]
**Risk**: [breaks existing callers? No/Low/Medium + mitigation]
**Before / After**: [minimal code block per scoring.md §7]

### Phase 2: <next scope> (PR count: 1–2)
[Same structure]

### Architecture (after)
[Mermaid graph showing the target shape, bad edges removed]

## Alternatives Considered
### Alternative A: <approach>
- Why not: [reason]
### Alternative B: <approach>
- Why not: [reason]

## Safety Checklist
- [ ] Deepens existing modules (not create new ones)
- [ ] No new layers/abstractions without justification
- [ ] Each phase is one PR-sized chunk
- [ ] Phases independent, or order is stated
- [ ] Reduces change amplification
- [ ] Callers' code becomes simpler
- [ ] No config/flags added to defer hard decisions

## Merge Order & Dependencies
[Dependencies between phases, or "independent — any order"]

## Success Metrics
[How you'll know it helped — e.g. "new features add 1 file, not 3+"; Health Score rises]

## Estimated Effort
[Overall scope: small / medium / large (PR count), with reasoning. No durations.]
```

---

## Finding template (used in all three commands)

Every finding, regardless of command, follows the diagnostic chain from
`rules/scoring.md` §3:

```
**<Red Flag Name>** — `<file:line or module>`
[red-flag: <id> · <chapter file> · Confidence: <High/Medium/Low> · Effort: <S/M/L>]
- Symptom: [observable evidence]
- Root cause: [the design decision or omission that produced it]
- Consequence: [maintainability cost; which follow-up red flag it triggers]
- Fix: [smallest reviewable change, or explicit defer-to-follow-up]
```

Worked example:

```
**Information Leakage** — `service.py:45-60`
[red-flag: information_leakage · 02-deep-modules.md · Confidence: High · Effort: S]
- Symptom: 呼び出し側が models.User と UserStatus.PENDING を直接 import している
- Root cause: User の状態判定をリポジトリに置かず、呼び出し側に enum を露出した
- Consequence: status enum を変えると全呼び出し側に波及（change_amplification を誘発）
- Fix: repository に is_pending() を定義し、呼び出し側は enum を知らずに済むようにする
```

---

## Language Specifics

### For PR Review Comments

When writing comments to post in a PR, use the matching template from `templates/`:

- `templates/pr-comments-en.md` for English-speaking teams
- `templates/pr-comments-ja.md` for Japanese-speaking teams

The diagnostic chain (Symptom → Root cause → Consequence → Fix) maps onto the
状況 / 影響 / 提案 structure in those templates; keep the red-flag id and chapter
citation in the comment so the author can read the source chapter.

### For Refactoring Plans

Same format regardless of language; the proposal is technical and language-agnostic.

---

## Tone & Phrasing

### DO
- Be specific: "This information leakage causes a 5-file change when the User schema updates"
- Be actionable: "Move the status check into the repository"
- Assume good intent: "This pattern is common, but here's a deeper interface"
- Respect constraints: "Not blocking this PR, but worth a follow-up"
- Report scores and grades plainly, as a direction signal

### DON'T
- Use vague language: "This is bad design"
- Demand perfection: "This needs a complete rewrite"
- Ignore context: "Never use X" (without root cause)
- Overwhelm: 20 separate issues in one review (cap at the top findings)
- Be prescriptive without reason: "Refactor this to use a strategy pattern"
- Dramatize the score or use sensational framing
- State durations for effort (use PR count, never hours/days/weeks)

---

## When to Use "Blocking" vs "Non-Blocking"

### Blocking (must fix in this PR)
- Introduces new red flags not in the original code
- High Impact with Low Effort (easy fix, big payoff)
- Violates safety rules (broad rewrite, new layers, deferred decisions)
- Existing tests would fail or be misleading

### Non-Blocking (address later)
- Pre-existing red flags not made worse
- Would require broad changes (>1 PR / Effort L)
- Requires architectural decisions that should be separate
- Low friction; can refactor on its own timeline

### Follow-Up (optional, but recommended)
- Suggest a refactor plan or separate PR
- Include context: why it matters, and a small starting point
- Don't make it urgent; let the team prioritize
