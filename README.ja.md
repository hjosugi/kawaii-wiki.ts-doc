<!-- i18n: language-switcher -->
[English](README.md) | [日本語](README.ja.md)

# kawaii-wiki.ts ドキュメント

[公開ドキュメント](https://kawaii-wiki-ts-docs.up.railway.app/docs/home)で表示される公式記事の正本です。サイトへ接続できない場合も、[Markdown版の入口](content/docs/home.md)から全記事を参照できます。

## 記事を編集する

1. [`content/docs`](content/docs)から対象の`.md`ファイルを開きます。
2. 先頭のfrontmatter（`title`、`description`、`path`など）を残して本文を編集します。
3. `main`へコミットすると、通常5分以内にWikiへ取り込まれます。

Wiki画面で保存した変更も、このリポジトリへ自動でコミットされます。同じページをGitHubとWikiで同時に編集せず、一方の反映を確認してから次を編集してください。

## 保存されるもの

このリポジトリが保存するのはページ本文です。ユーザー、権限、履歴、検索インデックス、添付ファイルはRailwayのSQLiteと`/data`ボリュームにあるため、別途バックアップが必要です。

---

このリポジトリは、[公式ドキュメントサイト](https://kawaii-wiki-ts-docs.up.railway.app/docs/home)に表示されるMarkdownの正本です。公開サイトが利用できない場合は[Markdown版ドキュメント](content/docs/home.md)を参照してください。[`content/docs`](content/docs)以下のファイルを編集してください。`main`へのコミットは次回の同期時にWikiへ取り込まれ、通常5分以内に反映されます。Wikiでの編集も自動的にこちらへコミットされます。SQLiteとアップロードされたアセットは別途バックアップが必要です。
