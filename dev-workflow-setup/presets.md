# presets.md

このファイルはdev-workflow-setup SKILLの一部です。
プリセット定義（エージェント・ワークフロー・テンプレート・Hook・施策）を集約する。

---

## 8エージェント

| エージェント | モデル | ミッション | 型 |
|-------------|--------|-----------|-----|
| researcher | sonnet | Scout Agent起動、並列コンテキスト収集 | ワーカー |
| planner | opus | 実装計画作成（詳細設計）、ルール・アーキテクチャ整合チェック | オーケストレーター |
| developer | sonnet | コード実装 | ワーカー |
| tester | sonnet | テストケース作成・実行、エッジケース検証 | ワーカー |
| code-reviewer | sonnet | セキュリティ・性能・テストカバレッジ観点のレビュー | ワーカー |
| code-simplifier | sonnet | 同一ファイル5回修正時の自動起動、複雑度削減 | ワーカー |
| architecture | opus | システムアーキテクチャの設計・維持 | ワーカー |
| git-manager | sonnet | コミット・PR作成・ブランチ管理 | ワーカー |

---

## 8ステップワークフロー

```
計画フェーズ:     ①人間による要件準備 → ②Researcher → ③Planner
実装・検証フェーズ: ④Developer → ⑤Tester → ⑥Code Reviewer
完了処理フェーズ:  ⑦Doc Manager → ⑧Git Manager
```

---

## 5 Agent Team テンプレート

| テンプレート | パターン | リーダー | チームメート | 用途 |
|------------|---------|---------|------------|------|
| research | 並列リサーチ | planner (opus) | researcher x N (sonnet) | 異なる角度で同時調査→統合レポート |
| implement-orchestrator | 並列実装 | planner (opus) | developer x N (sonnet) | プラン分割・ファイル所有権分離で並列実装 |
| independent-impl | 独立実装 | planner (opus) | developer x N (sonnet) | worktreeで独立WS+PR作成、競合根本排除 |
| review | 並列レビュー | code-reviewer (opus) | code-reviewer x N (sonnet) | セキュリティ・性能・テスト同時レビュー |
| debug | 競合仮説デバッグ | developer (opus) | developer x N (sonnet) | 仮説→反証→根本原因特定 |

---

## 4 Hook

| イベント | ファイル名 | マッチャー | 機能 |
|---------|-----------|----------|------|
| SessionStart | session-init.cjs | — | ルール注入・セッション種別検出・Git情報取得 |
| UserPromptSubmit | dev-rules-reminder.cjs | — | 品質ガイドライン注入 |
| PreToolUse | descriptive-name.cjs | Write\|Edit | ファイル命名規則チェック |
| PostToolUse | post-edit-simplify.cjs | Edit | 同一ファイル5回修正→code-simplifier起動 |

---

## 3施策

| 施策 | 概要 | 追加生成物 |
|------|------|-----------|
| 1: 複数エージェントオーケストレーター | Agent Team + Sub Agent でタスク分割・並列実装 | team-*/SKILL.md, impl-orchestrator/SKILL.md, agent-team-orchestration/SKILL.md |
| 2: 経験則自動メモリ | memory-save-detector + skill-create-on-miss Hook で学習機会を自動検出 | memory-save-detector.cjs, skill-create-on-miss.cjs, .claude/memory/ |
| 3: ドキュメント自動メンテナンス | Scout Skill + docs-manager で800行超過自動分割 | scout/SKILL.md, docs/テンプレート |
