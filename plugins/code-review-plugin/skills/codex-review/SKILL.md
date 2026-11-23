---
name: codex-review
description: OpenAI Codex CLIを使用してコードレビューを実行する。Claude CodeからCodexにレビューを依頼したい時に使用。トリガー：「Codexにレビュー」「codexでコードレビュー」「差分をCodexでチェック」「Codexにコードを見てもらう」「レビューをCodexに依頼」
---

# Codex Review

Claude CodeからOpenAI Codex CLIにコードレビューを委譲するスキル。

## 前提条件

- Codex CLIインストール済み: `npm install -g @openai/codex`
- 認証済み: 初回`codex`実行時に認証
- **実行環境に関する注意**: `${CLAUDE_SKILL_DIR}/scripts/review.sh` は内部で `codex exec --dangerously-bypass-approvals-and-sandbox` を使用する（Docker 等でサンドボックスが動作しない環境でも `git diff` やファイル読み込みを自動実行するため）。承認確認とサンドボックスを無効化するため、信頼できるリポジトリや隔離環境での利用を前提とする

## レビューモード選択

| 状況 | モード | コマンド |
|------|--------|----------|
| コミット前チェック | `staged` | `${CLAUDE_SKILL_DIR}/scripts/review.sh staged` |
| 作業中の全変更確認 | `uncommitted` | `${CLAUDE_SKILL_DIR}/scripts/review.sh uncommitted` |
| 特定ファイル精査 | `file` | `${CLAUDE_SKILL_DIR}/scripts/review.sh file <path>` |
| 複数ファイル一括 | `files` | `${CLAUDE_SKILL_DIR}/scripts/review.sh files <path1> <path2>` |
| PR作成前 | `branch` | `${CLAUDE_SKILL_DIR}/scripts/review.sh branch <base> <target>` |
| コミット検証 | `commit` | `${CLAUDE_SKILL_DIR}/scripts/review.sh commit <hash>` |
| PRレビュー | `pr` | `${CLAUDE_SKILL_DIR}/scripts/review.sh pr <base_branch>` |

## 基本ワークフロー

1. レビュー対象を決定（差分/ファイル/ブランチ）
2. 適切なモードで `${CLAUDE_SKILL_DIR}/scripts/review.sh` を実行
3. Codexのフィードバックを確認
4. フィードバックをユーザーへ要約報告し、修正指示があれば対応する

## 使用例

```bash
# ステージング済み差分をレビュー
${CLAUDE_SKILL_DIR}/scripts/review.sh staged

# セキュリティ観点でレビュー
${CLAUDE_SKILL_DIR}/scripts/review.sh staged "Focus on security vulnerabilities"

# 特定ファイルをレビュー
${CLAUDE_SKILL_DIR}/scripts/review.sh file src/api/auth.ts

# feature/authブランチをmainと比較してレビュー
${CLAUDE_SKILL_DIR}/scripts/review.sh branch main feature/auth

# PRレビュー（origin/mainとの差分）
${CLAUDE_SKILL_DIR}/scripts/review.sh pr origin/main "Check for breaking API changes"
```

## 直接コマンド実行

スクリプトを使わず直接実行する場合（`${CLAUDE_SKILL_DIR}/scripts/review.sh` と同様に、Codex 自身に `git diff` の実行やファイル読み込みを行わせる形式）：

```bash
# ステージング済み差分をレビュー（標準サンドボックス環境）
codex exec "git diff --staged を実行して、ステージされた変更をレビューしてください。Report issues with severity, file, line range."

# 特定ファイルをレビュー（標準サンドボックス環境）
codex exec "ファイル src/main.ts を読み込んでレビューしてください。Report issues with severity, file, line range."

# Docker 等サンドボックスが動作しない環境（review.sh と同等の実行方法）
codex exec --dangerously-bypass-approvals-and-sandbox "git diff --staged を実行して、ステージされた変更をレビューしてください。Report issues with severity, file, line range."
```

## カスタム指示

全モードで最後の引数にカスタム指示を追加可能：

- `"Focus on security"` - セキュリティ脆弱性
- `"Check TypeScript types"` - 型安全性
- `"Review for performance"` - パフォーマンス
- `"Check accessibility"` - アクセシビリティ
- `"Verify error handling"` - エラーハンドリング

## 出力ファイル

レビュー結果は自動的に `.codex-reviews/` に保存される。

**ファイル名形式**: `{mode}_{context}_{timestamp}.md`

```
.codex-reviews/
├── staged_20231230_143052.md
├── branch_main-feature-auth_20231230_150123.md
├── file_src-api-auth-ts_20231230_151234.md
└── commit_abc1234_20231230_152345.md
```

**出力内容**:
- 日時、モード、コンテキスト
- カスタム指示（指定時）
- Codexのレビュー結果

**推奨**: `.gitignore` に `.codex-reviews/` を追加
