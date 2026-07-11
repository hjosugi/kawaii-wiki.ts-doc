---
title: Webパッケージ（英語原文）
description: Web UIの構成、開発、ビルド方法。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](web-package.md) | [日本語](web-package.ja.md)


> ソース: `../../web/README.md` — このページはリポジトリの英語原文です。

# kawaii-wiki.ts Web

kawaii-wiki.ts ハンズオンプロジェクトのための Vue 3 + Vite フロントエンド。

## これで学べること

- サーバーの `App` 型から Eden Treaty を使った型付きAPIコール
- Vueコンポーネントの状態管理とPiniaストア
- CodeMirrorを使ったMarkdown編集
- 検索、ページ閲覧、編集、ログイン、管理フロー
- サーバー送信イベントによるリアルタイムのページ変更更新
- 公開設定とCSS変数によるランタイムブランディング
- 公開設定からのホーム/ヘッダーのナビゲーション設定
- KaTeX CSS、Mermaid図、コンテンツタブのための遅延Markdown強化
- 新規ページ作成用の組み込みかつ永続化されたページテンプレート
- UIコードを薄く保ちつつ、ドメインルールは `@kawaii-wiki/core` に保持

## 実行方法

リポジトリのルートから:

```bash
bun install
bun run db:seed
bun run dev
```

`http://localhost:5180` を開きます。シード済みアカウントでサインインしてください:

```text
admin@example.com / password
```

## 便利なコマンド

```bash
bun --filter '@kawaii-wiki/web' dev
bun --filter '@kawaii-wiki/web' build
bun --filter '@kawaii-wiki/web' preview
bun --filter '@kawaii-wiki/web' typecheck
```

## 最初に読むべきファイル

| ファイル | 重要な理由 |
| --- | --- |
| `src/main.ts` | アプリのブートストラップとプラグイン設定 |
| `src/App.vue` | トップレベルのレイアウトとルートシェル |
| `src/lib/api.ts` | サーバーとの型付きクライアント契約 |
| `src/lib/branding.ts` | タイトル、ファビコン、カスタムCSS、信頼されたhead HTMLの適用 |
| `src/lib/markdownEnhance.ts` | コピー用ボタン、KaTeX CSS、Mermaid、タブでレンダリングされたMarkdownを強化 |
| `src/lib/pageTemplates.ts` | 組み込みのスターターテンプレートとカスタムテンプレートオプションヘルパー |
| `src/stores/auth.ts` | ログイン/セッション状態 |
| `src/views/PageEdit.vue` | Markdown編集フロー |
| `src/views/PageTemplatesView.vue` | エディター向けカスタムテンプレートマネージャー |
| `src/app.css` / `uno.config.ts` | テーマ変数とトークン対応ショートカット |

## 演習

1. 検索ビューにローディング状態と空状態を追加する。
2. 編集したページを保存するための小さなキーボードショートカットを追加する。
3. サーバーが対応したら、ページのリビジョン履歴を表示するルートを作成する。
4. API契約の型を保ちながらエディタープレビューを改善する。