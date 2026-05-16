# GitHub Actions ワークフロー ドキュメント

このドキュメントでは、`.github/workflows` ディレクトリに含まれる GitHub Actions ワークフローの設定について説明します。

---

## 1. Claude Code (`claude.yml`)

### 概要

イシューやプルリクエストのコメントで `@claude` とメンションすることで、Claude が自動的にタスクを実行するワークフローです。

### トリガー条件

以下のいずれかのイベントが発生し、かつ `@claude` が含まれている場合に実行されます。

| イベント | 条件 |
|---|---|
| `issue_comment` (created) | コメント本文に `@claude` が含まれる |
| `pull_request_review_comment` (created) | レビューコメント本文に `@claude` が含まれる |
| `pull_request_review` (submitted) | レビュー本文に `@claude` が含まれる |
| `issues` (opened, assigned) | イシュー本文またはタイトルに `@claude` が含まれる |

### 必要な権限 (permissions)

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
  actions: read
```

### 設定項目

| 設定キー | 必須 | 説明 |
|---|---|---|
| `claude_code_oauth_token` | ✅ | Claude Code の OAuth トークン。シークレット `CLAUDE_CODE_OAUTH_TOKEN` に設定する |
| `additional_permissions` | オプション | CI 結果を読み取るための追加権限 (`actions: read`) |
| `prompt` | オプション | Claude へのカスタムプロンプト。未指定の場合、コメントの内容が指示として使われる |
| `claude_args` | オプション | Claude CLI の追加引数（例: `--allowed-tools Bash(gh pr *)`） |

### 使い方

イシューやプルリクエストのコメント欄に `@claude` を含めてメッセージを送ると、Claude が自動的に応答・処理を行います。

```
@claude このバグを修正してください
@claude コードをレビューしてください
@claude テストを追加してください
```

---

## 2. Claude Code Review (`claude-code-review.yml`)

### 概要

プルリクエストが作成・更新されたときに、Claude が自動的にコードレビューを実行するワークフローです。

### トリガー条件

以下の PR イベント発生時に実行されます。

| イベント | 説明 |
|---|---|
| `opened` | PR が新規作成されたとき |
| `synchronize` | PR に新しいコミットがプッシュされたとき |
| `ready_for_review` | ドラフト PR がレビュー可能状態になったとき |
| `reopened` | クローズされた PR が再オープンされたとき |

> **オプション:** `paths` フィルターを設定することで、特定のファイル（例: `src/**/*.ts`）が変更された場合のみ実行するよう制限できます。

### 必要な権限 (permissions)

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
```

### 設定項目

| 設定キー | 必須 | 説明 |
|---|---|---|
| `claude_code_oauth_token` | ✅ | Claude Code の OAuth トークン。シークレット `CLAUDE_CODE_OAUTH_TOKEN` に設定する |
| `plugin_marketplaces` | ✅ | プラグインのソースリポジトリ URL |
| `plugins` | ✅ | 使用するプラグイン（`code-review@claude-code-plugins`） |
| `prompt` | ✅ | 実行するスラッシュコマンド（`/code-review:code-review <repo>/pull/<pr-number>`） |

### カスタマイズ例

PR の作成者でフィルタリングする場合は、ジョブの `if` 条件を有効にします。

```yaml
jobs:
  claude-review:
    if: |
      github.event.pull_request.user.login == 'external-contributor' ||
      github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR'
```

---

## 必要なシークレット設定

両ワークフローを使用するには、リポジトリの **Settings > Secrets and variables > Actions** に以下のシークレットを登録してください。

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の OAuth トークン（[Claude Code](https://claude.ai/code) から取得） |

---

## 参考リンク

- [claude-code-action リポジトリ](https://github.com/anthropics/claude-code-action)
- [使用方法ドキュメント](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [Claude Code CLI リファレンス](https://code.claude.com/docs/en/cli-reference)
