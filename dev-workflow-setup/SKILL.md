---
name: dev-workflow-setup
description: 開発ワークフロー基盤を構築するスキル。8エージェント・8ステップワークフロー・5 Agent Teamテンプレート・4 Hook・3施策をプリセットとして提供し、ヒアリングでプロジェクトに最適化した開発基盤を一括生成する。
argument-hint: ""
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent, AskUserQuestion
---

# 開発ワークフロー基盤構築スキル

開発ドメインに特化した基盤構築スキル。ハーネスエンジニアリング（AI + Tool + Rule + Hook + Skill の統合環境設計）の思想に基づき、プロジェクトに最適化された開発基盤を一括生成する。

## 生成物の全体像

```
org/
  README.md                              ← 組織図（Mermaid）+ ハーネス状況
  <agent>/CLAUDE.md                      ← 各エージェントの設計書
  workflows/
    sequential.md                        ← シングルエージェントWF定義
    <template>.md                        ← Agent Team WF定義

.claude/
  rules/
    role-<agent>.md                      ← 各エージェントの運用ルール（paths紐づき）
    project-rules.md                     ← プロジェクト固有ルール
  skills/
    ask-<agent>/SKILL.md                 ← 各エージェント呼び出しスキル
    team-<template>/SKILL.md             ← Agent Team ワークフロースキル
    scout/SKILL.md                       ← コンテキスト収集スキル（施策3）
    impl-orchestrator/SKILL.md           ← 実装オーケストレーションスキル（施策1）
    agent-team-orchestration/SKILL.md    ← エージェントチーム編成スキル（施策1）
  hooks/
    session-init.cjs                     ← SessionStart Hook
    dev-rules-reminder.cjs               ← UserPromptSubmit Hook
    descriptive-name.cjs                 ← PreToolUse Hook
    post-edit-simplify.cjs               ← PostToolUse Hook
    memory-save-detector.cjs             ← 学習シグナル検出（施策2）
    skill-create-on-miss.cjs             ← スキル自動昇格（施策2）
  memory/                                ← 経験則蓄積ディレクトリ（施策2）
  settings.json                          ← 差分マージ

docs/（施策3選択時）
  documentation-management.md            ← 開発ルールテンプレート
  system-architecture.md                 ← アーキテクチャテンプレート
```

## プリセット定義

プリセットの詳細は `presets.md` を参照。

## フロー

`AskUserQuestion` ツールを使い、インタラクティブな選択式でヒアリングを進める。
**すべてのステップを一度に提示せず、1ステップずつ進める。**

### AskUserQuestion の使い方

- 各質問は2〜4個の選択肢を用意する（「その他（自由入力）」は自動で追加されるため含めない）
- 複数選択が適切な質問には `multiSelect: true` を設定する
- 推奨する選択肢がある場合は先頭に置き「(推奨)」を付ける
- 1回の呼び出しで最大4問まで同時に聞ける
- 選択肢の `description` で補足説明を付け、ユーザーが判断しやすくする

### ステップ 0: 現状確認（自動実行）

スキル起動時に、まず以下を自動で確認する（ユーザーへの質問なし）:

- `org/**/CLAUDE.md`（Glob）、`.claude/hooks/`、`.claude/rules/`、`.claude/skills/`、`.claude/settings.json`、`.claude/memory/`、`docs/*.md` の存在を確認
- 技術スタックを自動推定（package.json→TS/JS、requirements.txt→Python、go.mod→Go、Cargo.toml→Rust、pom.xml→Java、composer.json→PHP、Gemfile→Ruby）

**分岐:**

- 既存の開発ハーネスが存在する場合: `AskUserQuestion` でモード選択（見直す→ステップ2、追加する→ステップ2、ハーネスだけ再生成→ステップ5、最初から作る→ステップ1）
- 存在しない場合: ステップ 1 から開始する

**何をしますか？** (multiSelect: false, header: "モード")
| label | description |
|-------|-------------|
| 見直す | 現在の構成を分析して改善提案する |
| 追加する | 不足エージェント・施策を追加する |
| ハーネスだけ再生成 | 既存のロールに対してハーネスのみ再生成する |
| 最初から作る | 既存の構成を無視して新規作成する |

---

### ステップ 1: プロジェクト情報の収集

`AskUserQuestion` で以下の3問を **1回の呼び出しで同時に**聞く。Step 0 で自動検出済みの場合は検出結果を表示してから質問し、該当選択肢に「(検出済み)」を付ける。

**Q1. プロジェクトの技術スタックは？** (multiSelect: true, header: "技術スタック")
| label | description |
|-------|-------------|
| TypeScript/JavaScript | Node.js, React, Next.js, Vue, Express 等 |
| Python | Django, FastAPI, Flask 等 |
| Go | Gin, Echo, net/http 等 |
| Java/Kotlin | Spring Boot, Gradle, Maven 等 |

**Q2. プロジェクトの規模感は？** (multiSelect: false, header: "規模")
| label | description |
|-------|-------------|
| 個人（1人） | 個人プロジェクト、学習・プロトタイプ |
| 小規模（2-3人）(推奨) | 小チームでのWebアプリ・API開発 |
| 中規模（4-10人） | 複数機能・複数サービスの開発 |

**Q3. リポジトリ構成は？** (multiSelect: false, header: "リポジトリ")
| label | description |
|-------|-------------|
| 単一アプリ(推奨) | 1つのアプリケーション |
| フロント/バック分離 | client/ と server/ に分離 |
| モノリポ | 複数サービスを1リポジトリで管理 |

回答をもとにプロジェクトプロファイルを整理し、テキストで確認を取る。

### ステップ 2: エージェント・ワークフロー選択

`AskUserQuestion` で以下の2問を **1回の呼び出しで同時に**聞く。推奨セットは規模に応じて自動調整（個人/小規模: 5エージェント、中規模: +git-manager, code-simplifier）。

**Q1. 有効にするエージェントは？** (multiSelect: true, header: "エージェント")
| label | description |
|-------|-------------|
| (推奨) researcher | Scout Agent。並列でプロジェクトdocsからコンテキスト収集 |
| (推奨) planner | 実装計画の作成（詳細設計）。ルール・アーキテクチャ整合チェック |
| (推奨) developer | コード実装。経験則メモリによる品質担保 |
| (推奨) tester | テストケース作成・実行。エッジケース検証 |
| (推奨) code-reviewer | セキュリティ・性能・テストカバレッジ観点のレビュー |
| code-simplifier | 同一ファイル5回修正時に自動起動。複雑度削減 |
| architecture | システムアーキテクチャの設計・維持 |
| git-manager | コミット・PR作成・ブランチ管理 |

**Q2. ワークフロー構成は？** (multiSelect: false, header: "ワークフロー")
| label | description |
|-------|-------------|
| シングルエージェント(推奨) | 1セッションで順にステップ実行。小規模タスク向き |
| マルチエージェント | Agent Teamで並列実装。大規模タスク向き |
| ハイブリッド | 通常はシングル、大タスク時はマルチに切替。両方のスキルを生成 |

---

### ステップ 3: 施策選択

3つの施策をテキストで概要説明した後、Q1とQ2を **1回の `AskUserQuestion` 呼び出しで同時に**提示する。施策1を選ばなかった場合はQ2の回答を無視する。

**Q1. 有効にする施策は？** (multiSelect: true, header: "施策")
| label | description |
|-------|-------------|
| (推奨) 施策1: 複数エージェントオーケストレーター | Agent Team + Sub Agent でタスク分割・並列実装。5つの Team テンプレート含む |
| (推奨) 施策2: 経験則自動メモリ | memory-save-detector + skill-create-on-miss Hook。学習機会を自動検出・Rules/Skill化 |
| 施策3: ドキュメント自動メンテナンス | Scout Skill + docs-manager。800行超過自動分割、コンテキスト最適化 |

**Q2. 使用する Agent Team テンプレートは？** (multiSelect: true, header: "Agent Team テンプレート")
| label | description |
|-------|-------------|
| (推奨) research | 異なる角度で同時調査し統合レポート作成 |
| (推奨) implement-orchestrator | プラン分割・ファイル所有権分離で並列実装 |
| independent-impl | worktreeで独立ワークスペース+PR作成。競合を根本排除 |
| (推奨) review | セキュリティ・性能・テスト等の観点で同時レビュー |
| debug | 仮説を立て互いに反証し根本原因を特定 |

### ステップ 4: プロジェクト固有ルールの収集

`AskUserQuestion` で以下の2問を **1回の呼び出しで同時に**聞く。Q1で「特になし」以外が選択された場合は各項目の詳細を自由入力で追加取得する。

**Q1. プロジェクト固有の開発ルールはありますか？** (multiSelect: true, header: "開発ルール")
| label | description |
|-------|-------------|
| コーディング規約 | ESLint, Prettier, Black 等の設定、または独自ルール |
| アーキテクチャ制約 | レイヤー分離、DI、特定パターンの使用義務等 |
| 命名規則 | ファイル名、関数名、変数名等のルール |
| 特になし | プロジェクト固有のルールはない |

**Q2. 参照ドキュメントの設定** (multiSelect: false, header: "ドキュメント")
| label | description |
|-------|-------------|
| テンプレートを生成(推奨) | documentation-management.md と system-architecture.md のテンプレートを生成 |
| 既に存在する | 既存のドキュメントパスを自由入力で指定 |
| 不要 | ドキュメント管理は行わない |

### ステップ 5: 確認と生成

全選択内容をサマリーテキストで提示する。生成物の全体像をツリー形式で表示する（エージェント・ワークフロー・施策・Hook・ルール・ドキュメント・生成ファイル一覧）。

**最終確認:** `AskUserQuestion` で「生成する / 修正したい」を確認する。「修正したい」の場合は修正を反映して再度ステップ 5 を表示する。

### ステップ 6: 一括生成 + 検証

生成・検証の詳細は `generation.md` を参照して実行する。

## 注意事項

- エージェント名（ディレクトリ名）は英語小文字のケバブケースを使う
- CLAUDE.md の内容は日本語で記述する
- 各エージェントの責任範囲が明確に分離されるよう設計する
- **ハーネス生成時は既存の settings.json を壊さないよう、差分マージを徹底する**
- **allowed-tools は最小権限の原則に従い、エージェントに不要なツールを含めない**
- **Hook 実装は `node -c` で構文チェックを通すこと**
- paths の glob パターンはプロジェクトの実際のディレクトリ構成に合わせて調整する
- 施策2の Hook は false positive（誤検知）を最小限にするため、検出パターンは保守的に設定する
