---
description: リポジトリ全体またはモジュール監査。設計問題を継続的に検出。aposd-core スキルを使う。
---

# aposd-review

リポジトリ全体またはモジュール単位で、APoSD の観点から設計上の問題を検出します。

PR ごとではなく、コードベース全体の設計状態を把握するのに使います。

## 対象の決め方

- 引数がない場合：
  - リポジトリ全体をスキャン
  - すべてのソースコードを対象（言語は Python / TypeScript / Go / Java など自動判定）
  - 例：`/aposd-review`

- ディレクトリが引数の場合：
  - 指定したディレクトリ以下をすべてスキャン
  - 例：`/aposd-review src/services`
  - 例：`/aposd-review ./api ./domain`（複数ディレクトリ指定可）

- ファイルが引数の場合：
  - 指定したファイルのみを対象
  - 例：`/aposd-review src/services/user.py`
  - 例：`/aposd-review main.go config.ts`（複数ファイル指定可）

- 混合指定：
  - ディレクトリとファイルを混在指定可
  - 例：`/aposd-review src/services src/api/handlers.py`

## 実行指針

必ず以下のスキルを使ってレビューしてください。

- `aposd-core` — red-flags、safety-rules、scoring、言語別プロフィールを参照
  - `rules/scoring.md` を必ず読み、Health Score・診断チェーン・優先度マトリクス・Mermaid 依存グラフを適用する

レビュー順序：

1. 対象範囲のすべてのソースファイルを走査
   - 言語別プロフィール（Python / TypeScript など）を自動適用
   - ファイルツリーの構造から設計レイヤーを推測
2. 各ファイル・モジュールで Red flag を検出し、診断チェーン（Symptom → Root cause → Consequence → Fix）で記述。red-flag id・APoSD 章引用・Confidence・Effort を付す（scoring.md §3,§4）
3. 相互ファイル間の依存・情報漏洩をクロスチェックし、Mermaid 依存グラフで赤旗モジュールを色分け（scoring.md §6）
4. Health Score とカウント分布を算出し、スコープ（ファイル数・行数・除外）を明記（scoring.md §1,§2）
5. Impact × Effort の優先度マトリクスで最優先 1 件を特定し、モジュール別に findings をグルーピング（scoring.md §5）
6. safety-rules に違反していないか（broad rewrite 兆候など）チェック
7. 改善提案はマイルド に、future refactor として提示。1 モジュールが 1 PR を超える場合は `/aposd-refactor-plan` に回す

## 出力形式

aposd-core の `rules/output-format.md`（`/aposd-review` 節）と `rules/scoring.md` に従ってください。レポートは次を必ず含みます。

1. **Design Health: N / 100**（grade・preset 明記）+ 重要度別カウント表 + スコープ（スキャンしたファイル数・行数・除外）
2. **Summary** — コードベース全体の設計状態を 2-4 行で。負債がどこに集中しているか
3. **Module Map** — Mermaid 依存グラフ。赤旗モジュールを赤（High）／黄（Medium）で色分けし、情報漏洩・pass-through の辺をラベル付け（単一ファイルのみのスコープでは省略）
4. **着手順位（Impact × Effort）** — 2×2 マトリクスと最優先 1 件
5. **Findings by Module** — モジュール別にグルーピング（最悪モジュールを先頭）。各赤旗を診断チェーンで記述

   ```
   **[Red Flag Name]** — `file:line_range`
   [red-flag: <id> · <章ファイル> · Confidence: <H/M/L> · Effort: <S/M/L>]
   - Symptom: [観測できる証拠]
   - Root cause: [そうなった設計判断・欠落]
   - Consequence: [保守性のコスト／誘発する赤旗]
   - Fix: [PR サイズの修正方向]
   ```
6. **Cross-Module Concerns** — ファイル間の設計問題（情報漏洩・変更波及・レイヤリング）。Module Map の辺を参照
7. **Safety Check** — 大幅な設計変更の必要性／broad rewrite 兆候／DDD・Clean Architecture 移行根拠を各 Yes/No + 理由
8. **Recommendation** — 改善の優先順位と次のステップ

## 出力言語

リポジトリの言語に応じて日本語または英語で返答します。

## 補足

- スタイル・lint の問題は対象外（`/code-review` に任せる）
- テスト不足は設計問題に関連する場合のみ言及
- パフォーマンスは設計問題に関連する場合のみ言及
- 既知の技術負債・TODO コメントがあれば参考情報として載せる
