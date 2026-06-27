# A Philosophy of Software Design — Claude Code 統合版

John Ousterhout の『A Philosophy of Software Design』をベースに Claude Code で実装。設計上の複雑性の原因を見つけ、その影響を評価し、安全に改善する手段を提供します。

## クイックスタート

このリポジトリで提供される 2 つのコマンド：

1. **`/aposd-pr-review`** — PR や差分をレビュー。浅いモジュール、情報漏洩、変更波及など設計の問題を指摘
2. **`/aposd-refactor-plan`** — リファクタの計画を立てる。コード生成はなし、段階的な改善戦略のみ

どちらも `aposd-core` というスキルを使用。Red flag の定義、安全ルール、Python/TypeScript の言語別テンプレートが含まれています。

### インストール

```bash
# Claude Code の設定ディレクトリにコピー
cp -r skills/aposd-core ~/.claude/skills/aposd-core
cp commands/aposd-*.md ~/.claude/commands/

# 動作確認
/aposd-pr-review
```

詳しくはこの下の「セットアップと使い方」を参照。

---

## セットアップと使い方

### インストール

#### 必要なもの
- Claude Code（CLI / デスクトップ app / web）
- コマンド機能に対応したバージョン
- Git リポジトリ

#### 方法 1: グローバルインストール（推奨）

```bash
# このリポジトリをクローンまたはダウンロード
git clone https://github.com/mori-dev/a-philosophy-of-software-design-skills.git
cd a-philosophy-of-software-design-skills

# Claude Code の設定にコピー
cp -r skills/aposd-core ~/.claude/skills/aposd-core
cp commands/aposd-*.md ~/.claude/commands/

# インストール確認
ls ~/.claude/commands/aposd-*.md  # 2 個のファイルが表示される
ls ~/.claude/skills/aposd-core/SKILL.md  # 存在することを確認
```

#### 方法 2: プロジェクト内にインストール

```bash
# プロジェクトリポジトリ内で実行
mkdir -p .claude/commands .claude/skills
cp -r <このリポジトリ>/skills/aposd-core .claude/skills/
cp <このリポジトリ>/commands/aposd-*.md .claude/commands/
```

インストール後、Claude Code で `/aposd-pr-review` と `/aposd-refactor-plan` が使えるようになります。

---

### コマンド 1: `/aposd-pr-review`

**何をするのか**: PR やコミット差分を APoSD の観点からレビュー。設計の問題を見つけ、それが重大か後回しでいいかを判定します。

**どのような場合に使うか**:
- 設計上重要な PR がマージ前に確認したい
- 変更がどの程度複雑性に影響するか理解したい
- 見落としがないか第二意見が欲しい

**使い方**:

```
/aposd-pr-review
                    # 現在の全ての変更をレビュー

/aposd-pr-review https://github.com/org/repo/pull/123
                    # GitHub PR を指定してレビュー

/aposd-pr-review src/services/user.py
                    # 特定のファイルをレビュー
```

**出力に含まれるもの**:
- **Summary**: 設計上の影響を簡潔に説明
- **Blocking Issues**: このコミット・PR で直すべき問題
- **Non-Blocking Observations**: 指摘するが、別の PR で直してもよい項目
- **Safety Check**: 大幅な設計変更の警告、DDD 勧告など
- **Recommendation**: 承認 / 変更要求 / 後続 PR 推奨

**検出する設計の問題**:
- Shallow module — インタフェースが実装ほどシンプルでない
- Information leakage — 呼び出し側が内部構造を知ってしまう
- Change amplification — 1 つの変更が複数ファイルに波及
- Vague naming、Special-case mixture、Excessive configuration
- Error-path complexity、Pass-through abstraction
- Temporal decomposition、Repetition、Nonobvious code

**推奨しないもの**:
- 単に長いから分割する
- 根拠なく DDD や Clean Architecture を勧める
- コード整理を名目に wrapper クラスを増やす
- 抽象層を無闇に増やす

---

### コマンド 2: `/aposd-refactor-plan`

**何をするのか**: リファクタの具体的な改善計画を立てる。何をどのような順序で直すか、各段階でのリスクはどうか、を段階的に示します。

**どのような場合に使うか**:
- 大きなリファクタに取り組む前に全体像を整理したい
- チームに改善計画を説明する必要がある
- 複雑な変更を小分けにして実装したい
- なぜこのモジュールが変更しづらいのか理解したい

**使い方**:

```
/aposd-refactor-plan src/services/user.py
                    # モジュールの改善計画を立てる

/aposd-refactor-plan error_handling
                    # テーマ・関心事で計画を立てる
```

**出力に含まれるもの**:
- **Root Cause Analysis**: 根本的な問題と、それがなぜ起きているか
- **Current State**: 今、何が うまくいっていて何が困っているか
- **Proposed Path**: 段階的な改善計画
  - 各段階は 1～2 個の PR にまとまるサイズ
  - ゴール、具体的な変更、テストへの影響
  - リスク評価と Before/After コード例
- **Alternatives**: なぜ別のアプローチを選ばなかったか
- **Success Metrics**: リファクタが成功したことをどう判断するか
- **Estimated Effort**: 小 / 中 / 大

**生成しないもの**:
- 実際のコードや patch
- 大規模な建築設計の変更
- デフォルトとして DDD/Clean Architecture を勧める
- 根拠なし新レイヤー

---

## 基本的な考え方

**ソフトウェア設計の中心的な課題は、複雑性をどう扱うかである。**

複雑性があると、システムを理解したり変更したりするのが難しくなります。複雑性を減らすために：

- 戦略的に考える（その場しのぎではなく）
- シンプルなインタフェースで強力な実装を隠す
- 内部の決定を外に漏らさない
- 特殊なケースを減らす
- 名前とコードで意図を明確にする
- きちんとした抽象化を使う

---

## Red Flag リファレンス

複雑性の警告サイン：

- **Shallow Module** — インタフェースが実装ほどシンプルでない
- **Information Leakage** — 呼び出し側が内部の決定を知る
- **Change Amplification** — 1 つの論理的変更が多数の場所に波及
- **Vague Naming** — 意図が関数名から不明
- **Special-Case Mixture** — 一般的ロジックと特殊ケースが混在
- **Excessive Configuration** — 難しい判断を defer する flag/option が多すぎる
- **Comments Repeat Code** — コメントが code の構文を再述
- **Unnecessary Error-Path Complexity** — try-catch が散乱する代わりに定義で排除すべき
- **Pass-Through Abstraction** — call を forward するだけのレイアー
- **Temporal Decomposition** — 実行順序に基づく構造分割
- **Repetition** — 同じロジックが複数箇所に重複
- **Nonobvious Code** — 理解に深い読み込みが必要

---

## Red Flag 定義と修正方法

完全なリファレンス：

- `skills/aposd-core/rules/red-flags.yaml` — 12 個の Red flag、症状、質問、修正方法
- `skills/aposd-core/profiles/python.md` — Python 言語別パターン
- `skills/aposd-core/profiles/typescript.md` — TypeScript 言語別パターン

---

## 安全ルール

`/aposd-pr-review` と `/aposd-refactor-plan` は Broad rewrite を防ぐ 9 つのルールを強制します：

1. **長いからという理由だけで分割しない** — 長さ単体は分割理由にならない
2. **レイアーをデフォルトで導入しない** — DDD/repository/service は情報隠蔽が本当に必要な場合のみ
3. **Wrapper クラスを安易に作らない** — コードを「整理する」ためだけにクラスを増やさない
4. **設定で難しい判定を defer しない** — 内部で判定を決める
5. **メッセージング/events で coupling を隠さない** — 直接 call が単純な場合が多い
6. **抽象レベルを増殖させない** — 各抽象は何かを隠すべき
7. **本当に壊れているか測定してからリファクタ** — 問題を理解してから
8. **新しいモジュールより既存モジュールを深くする** — 既存を deep にすることを優先
9. **エラーハンドリングを散乱させない** — 源で処理し、複数レイアーに渡さない

詳細は `skills/aposd-core/rules/safety-rules.md` 参照。

---

## PR コメント テンプレート

PR にコメントするときに使うテンプレート：

- `skills/aposd-core/templates/pr-comments-ja.md` — 日本語テンプレート
- `skills/aposd-core/templates/pr-comments-en.md` — 英語テンプレート

各 Red flag について Blocking（このPR で直す）と Non-blocking（後続でよい）バージョンを用意。

---

## 典型的なワークフロー：レビュー + 計画

### ステップ 1: PR 進行中にレビュー

```
/aposd-pr-review

# 出力：
# - Blocking issues（マージ前に直すべき）
# - Non-blocking observations（後続PR で直す）
# - Safety check 結果
```

### ステップ 2: 重大問題の場合、後続リファクタを計画

```
/aposd-refactor-plan src/services/user.py

# 出力：
# - Root cause 分析
# - マルチフェーズリファクタ計画
# - フェーズごとのリスク評価
# - Success metric
```

### ステップ 3: 計画に従い、PR サイズ単位で実装

計画の各フェーズに従う。各フェーズは 1 PR サイズ、reviewable で、独立して merge 可能。

---

## 言語対応

両コマンドはリポジトリまたはプロンプト言語に応じて日本語または英語で返答：

- `README.md` または `.claude/settings.json` が日本語なら日本語で応答
- そうでなければ英語

明示的に指定：

```
/aposd-pr-review --lang ja
/aposd-refactor-plan src/services/user.py --lang en
```

---

## 制限事項

- **高優先度バグ fix** — fix PR ではなく後続リファクタで使用
- **スコープ確定済み** — チームが「microservices に移行」と決定済みなら、この skill は異議を出してもそれに従わない
- **小さい単一ファイル変更** — Red flag 検出は repository/service/error handling レベルが最適。スタイル/小さいユーティリティ変更には向かない
- **コード生成** — 検出と計画のみ。コード生成や patch は出力しない

---

## ファイル構成

```
skills/aposd-core/
├── SKILL.md                      # Skill 説明と中核語彙
├── rules/
│   ├── red-flags.yaml           # 12 個の Red flag：症状、質問、修正
│   ├── safety-rules.md          # Broad rewrite 防止 9 ルール
│   └── output-format.md         # コマンド出力構造
├── templates/
│   ├── pr-comments-ja.md        # 日本語 PR コメント テンプレート
│   └── pr-comments-en.md        # 英語 PR コメント テンプレート
└── profiles/
    ├── python.md                # Python 言語別パターン
    └── typescript.md            # TypeScript 言語別パターン

commands/
├── aposd-pr-review.md           # /aposd-pr-review コマンド
└── aposd-refactor-plan.md       # /aposd-refactor-plan コマンド
```

---

## よくある質問

**Q: エディタでどう使う？**
A: Claude Code は VS Code、JetBrains IDE、web に統合。Claude sidebar で `/aposd-pr-review` と入力。

**Q: Red flag をカスタマイズできる？**
A: はい。`skills/aposd-core/rules/red-flags.yaml` を編集して `~/.claude/skills/aposd-core/` にコピー。

**Q: 推奨に同意しない場合は？**
A: この skill は提案。判断はあなたの。ただし safety rules（broad rewrite 防止）は厳密。Red flag 重要度と follow-up timing はコンテキスト次第。

**Q: `/code-review` とどう使い分ける？**
A: この skill は設計/建築。`/code-review` はコード品質・スタイル・テスト。両方で comprehensive review。

---

## 参考資料

- 原著: "A Philosophy of Software Design" by John Ousterhout（Yaknyam Press）
- Stanford CS 190（この考え方が開発された講座）
- 関連概念：情報隠蔽（David Parnas）、関心の分離
- 補完アプローチ：Clean Code、SOLID principles

---

## ライセンス

MIT（オリジナル APoSD ドキュメントに合わせる）

---

**今すぐ始める**: 次の PR で `/aposd-pr-review` を実行し、重大な問題が見つかったら `/aposd-refactor-plan` で計画。
