# Claude Code Skills

Claude Code で使えるカスタムSkills集です。対話形式でAIエージェント組織や開発ワークフロー基盤を設計・生成します。

## Skills 一覧

### org-chart

AIエージェントの組織設計を対話形式で行います。ヒアリングを通じてロール定義・ワークフロー・CLAUDE.md・rules を一括生成します。

**できること:**
- 目的に応じたロール構成の設計
- `.claude/rules/role-*.md` の生成
- `org/*/CLAUDE.md`（ミッション定義）の生成
- ワークフロー定義（`org/workflows/*.md`）の生成
- Agent Team テンプレートの生成

**使い方:**
```
/org-chart
```

### dev-workflow-setup

開発プロジェクト向けのワークフロー基盤を対話形式で一括生成します。8エージェント・8ステップワークフロー・Hooks をプリセットとして提供し、プロジェクトに最適化します。

**できること:**
- 開発ワークフローの設計（企画〜デプロイ）
- エージェント定義・ロール分担の生成
- Hooks（pre-commit, post-edit 等）の設定
- `.claude/settings.json` の生成

**使い方:**
```
/dev-workflow-setup
```

## インストール

Claude Code のプロジェクトで以下を実行してください:

```bash
# org-chart をインストール
claude install-skill https://github.com/worply-dev/claude-code-skills/tree/main/org-chart

# dev-workflow-setup をインストール
claude install-skill https://github.com/worply-dev/claude-code-skills/tree/main/dev-workflow-setup
```

## 設計思想

これらのSkillsは「たたき台を速く作る」ためのツールです。生成後のチューニング（ロール定義の微調整、禁止事項の追加、ワークフローの調整）は手動で行ってください。

設計パターンの詳細は以下の記事で解説しています:
- [Claude Codeで破綻しないCLAUDE.md設計 -- パターンと分割戦略](https://zenn.dev/ishiyan) (Zenn)
- [Worply Media](https://blog.worply.com/) -- AIツールの実践知を発信中

## 使ってみた方へ

Skills を実際に使ってみた感想・改善点があれば、ぜひ教えてください:
- [導入報告フォーム](https://docs.google.com/forms/d/e/1FAIpQLSdRTToQCOINn_v_cW-UUHXdTcxgi-hgnMAJ3xAFdpyf2qZ8dw/viewform)（3問・1分で完了）
- [GitHub Issues](https://github.com/worply-dev/claude-code-skills/issues) で報告
- X ([@kuni_p_](https://x.com/kuni_p_)) でメンション

## License

MIT
