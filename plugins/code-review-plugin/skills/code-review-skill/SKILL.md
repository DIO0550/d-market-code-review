---
name: code-review-skill
description: 言語共通のコードレビュースキル。コメント最適化、マジックナンバー検出、構造改善、防御的コード・フォールバックのレビューを実施。コードレビュー、PRレビュー、コード品質チェックを依頼された際に使用。
---

# Code Review Skill

言語共通のコードレビューを実施する。

## レビュー観点

| 観点 | 説明 | 参照 |
|------|------|------|
| コメント最適化 | Why/Docコメント以外を削除・置換 | [comment-review.md](references/comment-review.md) |
| マジックナンバー | 意味不明な数値・文字列リテラルの検出 | [magic-number-review.md](references/magic-number-review.md) |
| 構造改善 | Guard Clauses、Dead Code等の検出 | [structure-review.md](references/structure-review.md) |
| 防御的コード | 不要なフォールバック・「念のため」の防御的コードの検出 | [defensive-code-review.md](references/defensive-code-review.md) |

## 使用方法

1. レビュー対象のコードを受け取る
2. 該当する観点のreferenceを読み込む
   - **観点が指定されていない場合**: 全4観点のreferenceをすべて読み込み、指摘を整理して下記の「重要度分類」に統合して出力する
   - **特定の観点が指定された場合**: 該当するreferenceのみを読み込み、その形式に従って出力する
3. 各referenceの出力形式に従ってレビュー結果を出力する（複数観点を統合する場合は下記「重要度分類」に揃えて出力）

## 重要度分類

統合出力および `comment-review.md` / `defensive-code-review.md` では以下の3段階を使用する。`magic-number-review.md` および `structure-review.md` で定義されている優先度（高/中/低）との対応は以下の通り：

| 重要度分類 | 意味 | `magic-number-review.md` / `structure-review.md` との対応 |
|------------|------|-----------------------------------------------------------|
| **Blocking** | 即修正必須 | 優先度: 高（🔴 高優先度） |
| **Should Fix** | 修正推奨 | 優先度: 中（🟡 中優先度） |
| **Nice to Have** | 改善提案 | 優先度: 低（🟢 低優先度/許容） |
