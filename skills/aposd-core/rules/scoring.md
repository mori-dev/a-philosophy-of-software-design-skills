# Scoring & Enrichment: APoSD Review Commands

This document defines the quantification, prioritization, and visualization that
all three commands (`/aposd-review`, `/aposd-pr-review`, `/aposd-refactor-plan`)
add on top of a plain red-flag list. The goal is a report that is measurable,
prioritized, and traceable to a specific APoSD chapter — not a flat bullet list.

All numbers are **deterministic**: the same finding set always produces the same
score and the same quadrant placement.

---

## 1. Design Health Score (0–100)

Start at 100 and deduct per finding by severity. Floor at 0, round to integer.

| Severity | balanced (default) | strict | legacy-friendly |
|----------|-------------------|--------|-----------------|
| 🔴 High   | −12               | −18    | −8              |
| 🟡 Medium | −5                | −8     | −3              |
| 🟢 Low    | −2                | −3     | −1              |

`Score = max(0, round(100 − Σ deductions))`

Use **balanced** unless the user asks for a stricter or more lenient preset.
State which preset was used.

### Grade bands

| Score  | Grade | Meaning                                            |
|--------|-------|----------------------------------------------------|
| 90–100 | A     | Healthy. Deep modules, little leakage.             |
| 75–89  | B     | Minor design debt. Address in normal flow.         |
| 60–74  | C     | Noticeable debt. Plan a refactor for the worst.    |
| 40–59  | D     | Significant debt. Refactor before adding features. |
| 0–39   | E     | High change amplification. Refactor-plan first.    |

The score is a relative signal for tracking direction over time, not an absolute
verdict. Do not dramatize it; report it plainly.

---

## 2. Severity & coverage block

Every report opens with a count distribution and an explicit scope statement, so
the reader knows both the result and what was (and was not) examined.

```
## Design Health: 62 / 100 (C, balanced preset)

| Severity | 件数 | 該当 red-flag                                  |
|----------|-----|-----------------------------------------------|
| 🔴 High   | 3   | shallow_module ×2, change_amplification ×1     |
| 🟡 Medium | 5   | vague_naming ×3, cognitive_load ×2             |
| 🟢 Low    | 2   | comments_repeat_code ×2                         |

スコープ: 47 ファイル / 8,200 行をスキャン（除外: tests/, *.generated.ts）
```

For `/aposd-pr-review`, scope is the diff: state the base ref, file count, and
added/removed line counts.

---

## 3. Diagnostic chain (4 steps + citation)

Replace the old `Problem / Risk / Suggestion` shape with a fixed 4-step chain.
Each finding carries its `red-flags.yaml` id, an APoSD chapter citation, and
`Confidence` / `Effort` tags. This is the unit of every finding in every command.

```
**Shallow Module** — `UserRepository` (repo.py:12-40)
[red-flag: shallow_module · 02-deep-modules.md · Confidence: High · Effort: S]
- Symptom: 6 メソッドが 1 行の SQL ラッパー。インタフェースが実装と同じ深さ
- Root cause: ORM を薄く包んだだけで、キャッシュ・検証・整形を呼び出し側に残した
- Consequence: User schema 変更が呼び出し側 4 ファイルに波及（change_amplification を誘発）
- Fix: find(**filters) に集約し SQL/整形を内部に隠蔽（before/after は下記）
```

- **Symptom**: observable evidence in the code (what you can point at).
- **Root cause**: the design decision or omission that produced it ("why it happened").
- **Consequence**: what it costs for maintainability, naming any follow-up red flag it triggers.
- **Fix**: the smallest reviewable change, or an explicit defer-to-follow-up.

### Confidence

- **High** — the pattern is unambiguous in the code shown.
- **Medium** — likely, but depends on context not fully visible.
- **Low** — a hypothesis worth checking; say what would confirm or refute it.

### Effort (PR-sized, never time-based)

- **S** — one reviewable PR (<30 min review).
- **M** — 1–2 PRs.
- **L** — multi-PR; route to `/aposd-refactor-plan` instead of fixing inline.

Do **not** state durations (hours/days/weeks). Effort is PR count only.

---

## 4. Red-flag → APoSD chapter citation map

Cite the chapter the finding is grounded in. This is what distinguishes these
commands from generic linters: every finding traces back to a chapter in this repo.

| red-flag id                          | Chapter file                  |
|--------------------------------------|-------------------------------|
| shallow_module                       | 02-deep-modules.md            |
| information_leakage                  | 02-deep-modules.md            |
| overexposure                         | 02-deep-modules.md            |
| pass_through_abstraction             | 02-deep-modules.md            |
| change_amplification                 | 01-complexity-management.md   |
| cognitive_load                       | 01-complexity-management.md   |
| vague_naming                         | 04-naming-obviousness.md      |
| hard_to_pick_name                    | 04-naming-obviousness.md      |
| nonobvious_code                      | 04-naming-obviousness.md      |
| comments_repeat_code                 | 05-comments-documentation.md  |
| implementation_doc_contaminates_interface | 05-comments-documentation.md |
| hard_to_describe                     | 05-comments-documentation.md  |
| unnecessary_error_path_complexity    | 03-error-handling.md          |
| special_case_mixture                 | 06-general-purpose-design.md  |
| conjoined_methods                    | 06-general-purpose-design.md  |
| excessive_configuration              | 06-general-purpose-design.md  |
| temporal_decomposition               | 06-general-purpose-design.md  |
| repetition                           | 06-general-purpose-design.md  |

If a finding spans two chapters, cite the primary one and mention the second in
the Root cause line.

---

## 5. Priority matrix (Impact × Effort)

Plot every finding on a 2×2 of design Impact against fix Effort, then name the
single first action. This turns the severity list into an ordered worklist.

- **Impact** = severity, raised one level if the finding's `follow_up_issues` /
  `related_flags` show it propagates to other modules (high spread).
- **Effort** = the S / M / L from each finding.

```
## 着手順位（Impact × Effort）

         Effort 低 (S) ───────────────► 高 (L)
Impact 大 │ ① service.py 情報漏洩        ④ レイヤ全体の再編
         │   今すぐ・1PR                  → refactor-plan へ
Impact 小 │ ② vague naming ×3            ③ comment 整理
         │   ついでに                     後回し可

最優先: ① service.py:45-60 情報漏洩（High impact × S effort）
```

Rule: the **top action** is the finding with the highest Impact at the lowest
Effort. If several tie, pick the one with the widest spread (most follow-up flags).

---

## 6. Mermaid dependency graph (module-level reports)

For `/aposd-review` (and the architecture view of a refactor plan), render a
module dependency graph and color modules that carry red flags. The graph makes
information-leakage and pass-through edges visible at a glance.

```mermaid
graph LR
  controller --> service
  service -.->|pass-through 透過のみ| repository
  controller -->|UserStatus を直 import: 情報漏洩| models
  repository --> db
  classDef bad fill:#fdd,stroke:#c00,color:#900;
  classDef warn fill:#ffd,stroke:#cc0,color:#660;
  class models bad;
  class service warn;
end
```

Conventions:
- Solid edge = a real call/dependency. Dashed edge = a pass-through or a leak; label it with the red flag.
- `bad` (red) = a module with a 🔴 High finding. `warn` (yellow) = 🟡 Medium. Leave clean modules unstyled.
- Keep the graph to the modules in scope; do not draw the whole repo if only a directory was reviewed.

Skip the graph when the scope is a single file (there is no inter-module shape to show).

---

## 7. Before / After code block

Every blocking finding (and every refactor-plan phase) shows a minimal
before/after. Seed it from the matching `example_wrong` / `example_right` in
`red-flags.yaml`, then adapt to the actual code under review. Keep it short — just
enough to show the interface getting deeper, not a full rewrite.

```python
# Before — caller orchestrates internal SQL
users = repo.get_active_users()        # 1行SQLラッパー
verified = [u for u in users if u.status == UserStatus.PENDING]

# After — interface does more, caller does less
verified = repo.find(status="pending", verified=True)
```

---

## 8. Avoiding false positives (calibration)

A review is only trusted if its findings are real. Before reporting any finding:

- **Check `not_a_red_flag_when`** in `red-flags.yaml`. If the code matches one of
  those cases, do not report it. The book is explicit that every principle can be
  overdone; over-flagging is itself a design-review failure.
- **Flag only the avoidable instance.** Cross-cutting changes, required interface
  implementations, and domain-inherent exposure are not defects.
- **Weight severity by exposure.** The same pattern on a public API or a hot path
  is higher Impact than on internal, rarely-touched code. Raise or lower severity
  accordingly rather than using the flag's default blindly.
- **Use Confidence honestly.** `hard_to_pick_name` and `hard_to_describe` are hints,
  not defects — report them at Confidence: Low and frame the Fix as "consider",
  not "must". Reserve High for unambiguous evidence in the code shown.
- **Cap the report.** Prefer the top findings (per the priority matrix) over an
  exhaustive list. 20 low-value findings bury the few that matter.

When in doubt whether something is a real finding, lower its Confidence and say
what would confirm it, rather than dropping it or overstating it.

## Tone

Report scores, grades, and graphs in plain, factual terms. Do not use sensational
or childish framing for findings or scores. The enrichment exists to make the
report measurable and ordered, not dramatic.
