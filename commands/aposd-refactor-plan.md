---
description: リファクタ計画を APoSD 観点で策定。コード変更なし、戦略のみ。PR サイズに分割。aposd-core スキルを使う。
---

# aposd-refactor-plan

リファクタをコード変更せず計画します。根本的な複雑性問題を特定し、PR サイズの段階的な進め方を提案します。

## 対象の決め方

- 引数がある場合：
  - `$ARGUMENTS` をモジュール名またはフィーチャー名として扱う
  - 例：`/aposd-refactor-plan src/services/user.py`
  - 例：`/aposd-refactor-plan user authentication flow`
- 引数がない場合：
  - ユーザーにモジュール名またはフィーチャー名の入力を求める（または最近変更されたモジュールを提案）

## 実行指針

必ず以下のスキルを使ってください。

- `aposd-core` — red-flags、safety-rules、scoring、出力形式を参照
  - `rules/scoring.md` を必ず読み、Health Score・診断チェーン・Mermaid 依存グラフを適用する

計画策定の順序：

1. 対象モジュール / フィーチャー を理解
2. red-flags.yaml から主要な赤旗を特定
3. Root Cause を診断チェーン（Symptom → Root cause → Consequence）で分析し、red-flag id・APoSD 章引用・Confidence を付す（scoring.md §3,§4）
4. 現在の状態（何が機能しているか、何が壊れているか）を文書化し、現状アーキテクチャを Mermaid 依存グラフで図示（scoring.md §6）
5. 現在の Design Health を算出し、各 Phase 完了後の projected score を見積もる（scoring.md §1）
6. 修正の段階的パス を設計（各 Phase は 1-2 PR 分）。各 Phase に before/after を付す（scoring.md §7）
7. safety-rules をチェック（split without cause、layer defaults、configuration deferral なし）
8. 各 Phase の risk と test impact を評価し、目標アーキテクチャを Mermaid で図示
9. 最後に Success Metrics を定義

## Output Format

```
# APoSD Refactoring Plan: [Module/Feature]

## Target
- File(s): [対象ファイル]
- Scope: [問題のある側面]
- Design Health (current): N / 100 (grade) — projected after plan: N

## Root Cause Analysis

### Primary Red Flag: [Name]
[red-flag: <id> · <章ファイル> · Confidence: <H/M/L>]
- Symptom: [観測できる証拠]
- Root cause: [そうなった設計判断・欠落]
- Consequence: [保守性のコスト／誘発する赤旗]

### Secondary Red Flags
[関連する赤旗と、Primary との関係]

## Current State

### Architecture (before)
[Mermaid 依存グラフ。現状の形と問題のある辺を図示]

### What Works Well
[Aspects to preserve]

### What's Broken
[Specific friction points; e.g., PR to add feature X required 5-file changes]

## Proposed Path

### Phase 1: [Scope Name] (PR count: 1-2)
**Goal**: [Specific, measurable change]
**Health delta**: N → N（この Phase 完了後）
**Changes**:
- [Concrete change 1]
- [Concrete change 2]
**Test Impact**: [Which tests update; why]
**Risk**: [Will this break existing callers? No/Low/Medium with mitigation]
**Before/After**: [改善を示す最小コード例]

### Phase 2: [Next Scope] (PR count: 1-2)
[Same structure]

[Additional phases as needed]

### Architecture (after)
[Mermaid 依存グラフ。目標の形。問題のある辺が消えている]

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
[修正がうまくいったことをどう測定するか]

## Estimated Effort
[全体スコープ：small / medium / large とその根拠]
```

## 出力形式

aposd-core の output-format.md を参照してください。

## 出力言語

リポジトリまたはプロンプトの言語に応じて日本語または英語で返してください。

## 補足

- このコマンドはコード変更を生成しません。計画のみ
- 実装は別途、計画に従って実行する
- 各 Phase が 1-2 PR に収まることを確認
- Merge 順序が依存しないか、依存の場合は明記
