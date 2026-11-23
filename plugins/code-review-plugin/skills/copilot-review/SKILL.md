---
name: copilot-review
description: GitHub Copilot CLIを使用してコードレビューを実行する。Claude CodeからCopilotにレビューを依頼したい時に使用。トリガー：「Copilotにレビュー」「copilotでコードレビュー」「差分をCopilotでチェック」「Copilotにコードを見てもらう」「レビューをCopilotに依頼」
---

# Copilot Review

Claude CodeからGitHub Copilot CLIにコードレビューを委譲するスキル。

## 前提条件

- GitHub Copilot CLI インストール済み
- 認証済み: 初回 `copilot` 実行時に認証

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
3. Copilotのフィードバックを確認
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

スクリプトを使わず直接実行する場合（`${CLAUDE_SKILL_DIR}/scripts/review.sh` と同様に、標準入力からパイプで渡す形式）：

```bash
# 差分を標準入力からパイプで渡す
git diff --staged | copilot -p "Review this code diff. Report issues with severity, file, line range."

# ファイル内容を標準入力からパイプで渡す
cat src/main.ts | copilot -p "Review this TypeScript file for issues. Report issues with severity, file, line range."
```

## カスタム指示

全モードで最後の引数にカスタム指示を追加可能：

- `"Focus on security"` - セキュリティ脆弱性
- `"Check TypeScript types"` - 型安全性
- `"Review for performance"` - パフォーマンス
- `"Check accessibility"` - アクセシビリティ
- `"Verify error handling"` - エラーハンドリング

## 出力ファイル

レビュー結果は自動的に `.copilot-reviews/` に保存される。

**ファイル名形式**: `{mode}_{context}_{timestamp}.md`

```
.copilot-reviews/
├── staged_20231230_143052.md
├── branch_main-feature-auth_20231230_150123.md
├── file_src-api-auth-ts_20231230_151234.md
└── commit_abc1234_20231230_152345.md
```

**出力内容**:
- 日時、モード、コンテキスト
- カスタム指示（指定時）
- Copilotのレビュー結果

**推奨**: `.gitignore` に `.copilot-reviews/` を追加
