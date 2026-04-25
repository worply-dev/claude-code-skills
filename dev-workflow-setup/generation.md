# generation.md

このファイルはdev-workflow-setup SKILLの一部です。
ステップ6「一括生成 + 検証」の詳細手順を定義する。

---

## ステップ 6: 一括生成 + 検証

承認後、以下の順序で生成する。並列で生成可能なファイルは Agent を活用して並列生成する。

### 生成順序

1. `org/<agent>/CLAUDE.md` — 各エージェントの設計書
2. `org/workflows/*.md` — ワークフロー定義
3. `org/README.md` — 組織図
4. `.claude/rules/role-<agent>.md` — 運用ルール
5. `.claude/rules/project-rules.md` — プロジェクト固有ルール（入力があった場合）
6. `.claude/skills/ask-<agent>/SKILL.md` — 個別呼び出しスキル
7. `.claude/skills/team-<template>/SKILL.md` — Agent Team スキル（施策1選択時）
8. `.claude/skills/scout/SKILL.md` — Scout スキル（施策3選択時）
9. `.claude/skills/impl-orchestrator/SKILL.md` — 実装オーケストレーションスキル（施策1選択時）
10. `.claude/skills/agent-team-orchestration/SKILL.md` — チーム編成スキル（施策1選択時）
11. `.claude/hooks/*.cjs` — Hook 実装ファイル
12. `.claude/memory/` — ディレクトリ作成（施策2選択時）
13. `docs/*.md` — 参照ドキュメントテンプレート（選択時）
14. `.claude/settings.json` — 設定の差分マージ（**必ず最後に実行**）

### 生成後の自動検証

1. 各 `org/<agent>/CLAUDE.md` が存在し、ミッション・責任範囲・実行プロトコルを含むか
2. 各 `.claude/rules/role-<agent>.md` が存在し、paths が正しいか
3. 各 `.claude/skills/*/SKILL.md` が存在し、allowed-tools が設定されているか
4. `.claude/hooks/*.cjs` が存在し、`node -c` で構文チェックが通るか
5. `.claude/settings.json` の JSON が valid か、hooks 設定が正しいか
6. `org/README.md` の Mermaid 図が選択した全エージェントを含むか
7. 施策1選択時: Agent Team スキルと対応する workflow 定義の整合性
8. 施策2選択時: `.claude/memory/` ディレクトリの存在

検証結果をテキストで報告し、問題があれば自動修正する。

---

## 生成テンプレート

(Templates T1-T10 are included in the full SKILL.md)
