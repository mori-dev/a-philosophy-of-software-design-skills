---
description: APoSD 設計レビュー。現在の diff を APoSD red flags の観点でレビュー。コード変更は提案しない。aposd-core スキルを必ず使う。
---

# aposd-pr-review

現在の diff（ステージ・未コミット変更）を A Philosophy of Software Design の観点からレビューします。

コード変更は提案しません。設計上の赤旗を検出し、ブロッキング（このPRで直す）か非ブロッキング（後続）かを区別して報告します。

## 実行指針

必ず以下のスキルを使ってレビューしてください。

- `aposd-core` — red-flags、safety-rules、scoring、出力形式を参照
  - `rules/scoring.md` を必ず読み、Health Score・診断チェーン・優先度マトリクスを適用する

レビュー順序：

1. 差分を取得
   - GitHub PR URL が指定されている場合：PR の差分を取得
   - ファイルパスが指定されている場合：そのファイルの変更を取得
   - 引数なしの場合：`git diff main...HEAD` （または `git diff master...HEAD`）で現在のブランチと main/master の差分を取得
2. 設計レベルでの赤旗を検出（shallow module、information leakage、change amplification など）
3. 各赤旗を診断チェーン（Symptom → Root cause → Consequence → Fix）で記述し、red-flag id・APoSD 章引用・Confidence・Effort を付す（scoring.md §3,§4）
4. Health Score とカウント分布を算出し、この差分が新たに増やした赤旗と元からあった赤旗を分けて集計（scoring.md §1,§2）
5. Impact × Effort の優先度マトリクスで最優先 1 件を特定（scoring.md §5）
6. 各フラグについて blocking / non-blocking を判定。blocking には before/after を付す（scoring.md §7）
7. safety-rules に違反していないか（broad rewrite、DDD勧め、wrapper濫造など）をチェック
8. 修正提案は小さく、reviewable に

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

aposd-core の `rules/output-format.md`（`/aposd-pr-review` 節）と `rules/scoring.md` に従ってください。レポートは次を必ず含みます。

1. **Design Health: N / 100**（grade・preset 明記）+ 重要度別カウント表 + スコープ（base ref・変更ファイル数・増減行数）+ この差分が増やした赤旗 vs 元からあった赤旗の内訳
2. **Summary** — 変更の設計的影響を 1-3 行
3. **着手順位（Impact × Effort）** — 2×2 マトリクスと最優先 1 件
4. **Blocking Issues** — 各赤旗を診断チェーンで記述し before/after を付す

   ```
   **[Red Flag Name]** — `file:line_range`
   [red-flag: <id> · <章ファイル> · Confidence: <H/M/L> · Effort: <S/M/L>]
   - Symptom: [観測できる証拠]
   - Root cause: [そうなった設計判断・欠落]
   - Consequence: [保守性のコスト／誘発する赤旗]
   - Fix: [このPRで実行可能な最小修正]

   Before / After:
   [最小のコード例]
   ```
5. **Non-Blocking Observations** — 同じ診断チェーン。Fix 行は follow-up を示す
6. **Safety Check** — 各項目に Pass/Fail + 一行理由
7. **Recommendation** — Approve / Request Changes / Approve with Follow-Up

## 出力言語

リポジトリまたはプロンプトの言語に応じて日本語または英語で返してください。

## 補足

- 設計観点のレビューなので、スタイル・lint は `/code-review` に任せる
- テストカバレッジ不足は設計問題に関連する場合のみ言及
- 赤旗が見つからなければ「設計観点での重大な指摘なし」と明記
