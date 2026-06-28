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

- `aposd-core` — red-flags、safety-rules、言語別プロフィールを参照

レビュー順序：

1. 対象範囲のすべてのソースファイルを走査
   - 言語別プロフィール（Python / TypeScript など）を自動適用
   - ファイルツリーの構造から設計レイヤーを推測
2. 各ファイル・モジュールで Red flag を検出
3. 相互ファイル間の依存・情報漏洩をクロスチェック
4. 重要度順に ranking
5. safety-rules に違反していないか（broad rewrite 兆候など）チェック
6. 改善提案はマイルド に、future refactor として提示

## 出力形式

aposd-core の output-format.md に準拠し、以下を含む：

### Summary
[コードベース全体の設計状態を簡潔に要約]

### Red Flags Found
[検出された設計問題を重要度順に列挙]

**[Red Flag Name]** — `file:line_range` または `directory/`
- Location: [ファイルまたはディレクトリ]
- Severity: High / Medium / Low
- Pattern: [見えるパターン]
- Impact: [このモジュール、または他モジュールへの影響]
- Suggestion: [改善方向（具体的な PR サイズの修正案）]

[複数の Red flag を繰り返す]

### Cross-Module Concerns
[ファイル間の設計問題]

- **Information Leakage**: `module_a.py` が `module_b.py` の内部を知っている
- **Change Amplification**: 関心が複数ファイルに散らばっている
- **Layering Issues**: レイアーの責務がぼんやりしている

### Safety Check
- 大幅な設計変更の必要性：[Yes / No]
- Broad rewrite の兆候：[Yes / No]
- DDD/Clean Architecture への移行検討の根拠：[有無]

### Recommendation
[改善の優先順位と次のステップ]

## 出力言語

リポジトリの言語に応じて日本語または英語で返答します。

## 補足

- スタイル・lint の問題は対象外（`/code-review` に任せる）
- テスト不足は設計問題に関連する場合のみ言及
- パフォーマンスは設計問題に関連する場合のみ言及
- 既知の技術負債・TODO コメントがあれば参考情報として載せる
