---
title: インラインコメントRFC（英語原文）
description: インラインコメント機能の設計案。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](inline-comments-rfc.md) | [日本語](inline-comments-rfc.ja.md)


> ソース: `docs/INLINE_COMMENTS_RFC.md` — このページはリポジトリの英語原文です。

# インラインアンカードコメントRFC

ステータス: ブロック/見出しアンカー付きコメントは実装可。任意のテキスト範囲は保留。

関連Issue: #258。

## 問題点

ページレベルのコメントは議論に便利ですが、レビュアーは特定の見出し、リスト項目、段落、または埋め込みブロックを指し示す必要がよくあります。Googleドキュメントのような範囲指定コメントシステムは高コストです。なぜならMarkdownの編集でオフセットが常に変わり、選択範囲が無効になる可能性があるためです。

## 決定

アンカードブロックコメントから開始します：

- 見出しのスラグ、フェンスブロックのID、または生成された段落/リスト項目のハッシュから導出される安定したブロックIDにアンカーを付ける。
- ページパス、アンカーID、オプションの引用プレビュー、通常のコメントスレッドIDを保存する。
- リーダーとエディタのプレビューでアンカーを小さなコメント表示としてレンダリングする。
- 編集後にアンカーが解決できない場合はページレベルのコメントにフォールバックする。

最初のバージョンでは任意の文字範囲選択は実装しません。これは機能の中で最もリスクが高いため、同じスレッドモデルの下で後から追加可能です。

## データモデルのスケッチ

`page_comments`の隣にテーブルを追加します：

```sql
CREATE TABLE page_comment_anchors (
  comment_id TEXT PRIMARY KEY,
  page_id TEXT NOT NULL,
  path TEXT NOT NULL,
  anchor_id TEXT NOT NULL,
  quote TEXT NOT NULL DEFAULT '',
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);
CREATE INDEX page_comment_anchors_page_idx ON page_comment_anchors(page_id);
CREATE INDEX page_comment_anchors_anchor_idx ON page_comment_anchors(path, anchor_id);
```

既存のコメント本文、作成者、解決状態、メンション処理は`page_comments`に残します。

## アンカー生成

- 見出し：Markdownレンダラーの既存のスラグ化された見出しIDを使用。
- フェンス：明示的な`id:`フィールドがあれば優先使用。なければフェンスタイプと正規化された本文のハッシュを使用。
- 段落/リスト項目：正規化されたテキストをハッシュ化し、`p-4f8a21`のような短いプレフィックスを付ける。
- レンダリング時に複数のブロックが衝突した場合は、発生回数のサフィックスを追加。

## UIの着地点

- リーダー：対象ブロックの右端にコメントマーカーを表示。
- エディタプレビュー：プレビューやビジュアル面に同じマーカーを表示。
- コメントパネル：まずアンカーごとにグループ化し、次に未解決/解決状態でグループ化。
- 空のアンカー状態：ブロックが消えた場合は引用と「ページレベルフォールバック」ラベルを表示し、スレッドを隠さない。

## 実装可否

現在のMarkdownの正準モデルに合致し、既存のコメントサービスを再利用できるため、ブロック/見出しバージョンは実装可。Markdownソース位置とレンダリングプレビューノード間の安定したマッピングがエディタに実証されるまでは、任意範囲は実装不可。