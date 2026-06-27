---
description: APoSD 設計レビュー。現在の diff を APoSD red flags の観点でレビュー。コード変更は提案しない。aposd-core スキルを必ず使う。
---

# aposd-pr-review

現在の diff（ステージ・未コミット変更）を A Philosophy of Software Design の観点からレビューします。

コード変更は提案しません。設計上の赤旗を検出し、ブロッキング（このPRで直す）か非ブロッキング（後続）かを区別して報告します。

## 実行指針

必ず以下のスキルを使ってレビューしてください。

- `aposd-core` — red-flags、safety-rules、出力形式を参照

レビュー順序：

1. 差分を取得
   - GitHub PR URL が指定されている場合：PR の差分を取得
   - ファイルパスが指定されている場合：そのファイルの変更を取得
   - 引数なしの場合：`git diff main...HEAD` （または `git diff master...HEAD`）で現在のブランチと main/master の差分を取得
2. 設計レベルでの赤旗を検出（shallow module、information leakage、change amplification など）
3. 重要度順に ranking
4. 各フラグについて blocking / non-blocking を判定
5. safety-rules に違反していないか（broad rewrite、DDD勧め、wrapper濫造など）をチェック
6. 修正提案は小さく、reviewable に

## 対象の決め方

- GitHub PR URL が引数の場合:
  - `$ARGUMENTS` に `https://github.com/org/repo/pull/123` のような PR URL がある場合、その PR の差分をレビュー
  - 例：`/aposd-pr-review https://github.com/mori-dev/myapp/pull/45`
- ファイルパスが引数の場合:
  - `$ARGUMENTS` に `src/services/user.py` のようなパスがある場合、そのファイル または ディレクトリをレビュー
  - 例：`/aposd-pr-review src/services/user.py`
- 引数がない場合:
  - 現在のブランチ（HEAD）と main ブランチ（存在しなければ master）との差分をレビュー
  - `git diff main...HEAD` または `git diff master...HEAD` を対象にする
  - 例：`/aposd-pr-review`

## レビュー方針

- Blocking issues：このPRで直すべき、またはブロックすべき設計問題
- Non-blocking observations：指摘するが、このPRでは修正不要。follow-up PR を示唆
- 各赤旗について、症状・影響・修正案を明記
- safety-rules 9 つ をチェック（split without cause、layer defaults、configuration deferral など）
- DDD/Clean Architecture は根拠なく推奨しない
- 既存モジュールを深くすることを優先

## 出力形式

aposd-core の output-format.md に従ってください。

### Summary
[変更の設計的影響を 1-3 行で要約]

### Blocking Issues
[このPRで直すべき赤旗]

各フラグについて：
- **[Red Flag Name]** — `file:line_range`
- Problem: [具体的な証拠]
- Risk: [なぜ重要か]
- Suggestion: [このPRで実行可能な修正]

### Non-Blocking Observations
[指摘するが、このPRでは不要]

各フラグについて：
- **[Red Flag Name]** — `file:line_range`
- Pattern: [見えるパターン]
- Follow-up: [後続PR のトピック]

### Safety Check
各チェック項目に Yes/No + brief reason

### Recommendation
Approve / Request Changes / Approve with Follow-Up

## 出力言語

リポジトリまたはプロンプトの言語に応じて日本語または英語で返してください。

## 補足

- 設計観点のレビューなので、スタイル・lint は `/code-review` に任せる
- テストカバレッジ不足は設計問題に関連する場合のみ言及
- 赤旗が見つからなければ「設計観点での重大な指摘なし」と明記
