このファイルはorg-chart SKILLの一部です。ステップ 5「ハーネス生成」の詳細を定義します。

---

# ステップ 5: ハーネス生成

org/ の CLAUDE.md 生成に加え、以下のハーネスを自動生成する。

---

## 5-1. Rules 生成（`.claude/rules/role-<name>.md`）

各ロールの運用ルールを paths 紐づきで生成する。

```markdown
---
paths:
  - "<このロールが操作する主要パスのglob>"
---

# <ロール名> 運用ルール

## このロールの責務
[CLAUDE.md のミッション・責任範囲から抽出]

## 操作対象
[このロールが読み書きするファイル・ディレクトリ]

## 判断基準
[分類・トリアージ等の具体的な判断ルール]

## 禁止事項
[このロールがやってはいけないこと]
```

**paths の決め方:**
- ロールの責任範囲から、操作対象のファイルパスパターンを特定する
- 例: 秘書 → `mail-digest/**`, `slack-digest/**`, `tasks.md`
- 例: リサーチャー → `mail-digest/graphic-notes/**`, `research/**`
- 例: エンジニア → プロジェクトのソースコードパス

---

## 5-2. Skills 生成（`.claude/skills/ask-<name>/SKILL.md`）

各ロールを呼び出すスキルを生成する。

```markdown
---
name: ask-<name>
description: <ロール名>として作業を実行する
argument-hint: "<依頼内容>"
allowed-tools: <ロールに必要なツールのみ>
---

# <ロール名>

あなたは「<ロール名>」として作業します。

## ロール定義
org/<name>/CLAUDE.md を読み込み、そのミッション・責任範囲に従って行動してください。

## 実行手順
1. まず `org/<name>/CLAUDE.md` を読む
2. 依頼内容を確認し、責任範囲内かチェックする
3. 責任範囲外の場合は、適切なロールを提案して終了する
4. 責任範囲内の場合は、タスクを実行する

## 完了報告
作業完了時は以下のフォーマットで報告する:
- 実施内容の概要
- 完了項目 / 未完了項目
- 申し送り事項（あれば）
```

**allowed-tools の決め方:**
- 各ロールのタスク内容から必要なツールを特定する
- 例: 秘書 → Gmail MCP, Slack MCP, Read, Write, Edit
- 例: リサーチャー → WebSearch, WebFetch, Gemini MCP, Read, Write
- 例: エンジニア → Bash, Read, Write, Edit, Glob, Grep, Agent
- 不要なツールを含めない（最小権限の原則）

---

## 5-3. ワークフロー定義生成（`org/workflows/<name>.md`）

ステップ 3.5 で決定した各ワークフローの実行パターンを定義ファイルとして保存する。
これはドキュメントであると同時に、チーム実行スキルの参照元になる。

```markdown
# <ワークフロー名>

## 実行パターン
SubAgent / Agent Team

## 参加ロール

| ロール | 役割 | モデル |
|--------|------|--------|
| PM | リーダー / オーケストレーター | opus |
| エンジニア | チームメート / ワーカー | sonnet |

## 実行フロー

### SubAgent の場合
1. PM が秘書・リサーチャー・SW に並列委譲
2. 各ワーカーが結果を返す
3. PM が統合して報告

### Agent Team の場合
1. PM（リーダー）がチームメートをスポーン
2. PM → broadcast: 対象と観点を共有
3. メンバー間で message をやり取りしながら作業
4. 各メンバー → PM: 結果報告
5. PM → broadcast: 最終判断を共有

## コンフリクト防止
[ファイル編集の担当分け等]
```

---

## 5-4. Agent Team スキル生成（`.claude/skills/team-<workflow>/SKILL.md`）

Agent Team パターンのワークフローには、チーム実行用のスキルを生成する。

````markdown
---
name: team-<workflow>
description: <ワークフロー名>を Agent Team で実行する
argument-hint: "<対象や目的>"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent, AskUserQuestion
---

# <ワークフロー名>（Agent Team）

## チーム構成
- **リーダー**: <ロール名> (`<model>`)
- **チームメート**: <ロール名> (`<model>`), <ロール名> (`<model>`)

## 実行手順

1. `org/workflows/<workflow>.md` を読み込む
2. リーダーのロール定義（`org/<role>/CLAUDE.md`）を読み込む
3. 以下の指示でチームメートをスポーンする:

### チームメート起動プロンプト

各チームメートに対して Agent ツールで以下を指示する:

```
あなたは「<ロール名>」としてチームに参加します。
ロール定義: org/<role>/CLAUDE.md

## 今回のタスク
[ワークフローに応じた具体的な作業内容]

## 他メンバーとの連携
- リーダー (<ロール名>) に進捗を報告する
- <他メンバー> と技術的な確認を直接やり取りする

## モデル
<model>

## 完了報告
作業完了時はリーダーに以下を報告:
- 実施内容の概要
- 完了項目 / 未完了項目
- 他メンバーへの申し送り
```

## コスト見積もり目安
- リーダー: <model> × 1セッション
- チームメート: <model> × N セッション
- **合計 context 数: N+1**（SubAgent の約 N+1 倍のコスト）
````

---

## 5-5. Hooks・Settings 追記（`.claude/settings.json`）

既存の settings.json に**マージ**する（既存設定を上書きしない）。

**Agent Team 有効化（Agent Team パターンが1つ以上ある場合）:**
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**生成する Hooks:**

**SessionStart（セッション開始時）:**
- NEXT-SESSION.md の表示
- 未対応タスク数の表示
- 組織構成のサマリー表示

**PreToolUse（ツール実行前のガードレール）:**
- 保護ディレクトリ（`.credentials/`, `.private/`）への書き込みブロック
- ロールの責任範囲外のファイル操作への警告（オプション）

**既存の hooks がある場合:**
- 同じイベント・マッチャーの hook は上書きせず、配列に追加する
- 設定前に現在の settings.json を読み取り、差分マージする

**マージルール（差分マージ手順）:**
1. `.claude/settings.json` を Read で読み取る
2. 既存キーを保持しつつ追加キーのみ追記する
3. hooks は同一イベント・同一コマンドの重複を避けつつ配列に追加する
4. 既存の hooks を削除しない

> Agent Team パターンを使わない場合は `env` ブロックを省略する。

---

## 5-6. 生成後の検証

すべての生成が完了したら、自動で以下を検証する:

1. 各 `org/<role>/CLAUDE.md` が存在し、必須セクションを含むか
2. 各 `.claude/rules/role-<name>.md` が存在し、paths が正しいか
3. 各 `.claude/skills/ask-<name>/SKILL.md` が存在し、allowed-tools が設定されているか
4. Agent Team パターンがある場合:
   - `org/workflows/<name>.md` が存在するか
   - `.claude/skills/team-<name>/SKILL.md` が存在するか
   - `settings.json` に `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` が設定されているか
   - 各チームメートのモデル指定が記載されているか
5. `.claude/settings.json` の JSON が valid か
6. `org/README.md` の Mermaid 図が全ロールを含むか
7. `.claude/hooks/session-init.cjs` が存在し、`node -c .claude/hooks/session-init.cjs` で構文チェックが通るか

検証結果をテキストで報告し、問題があれば修正する。
