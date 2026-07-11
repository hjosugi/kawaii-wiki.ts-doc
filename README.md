# kawaii-wiki.ts documentation

[公開ドキュメント](https://kawaii-wiki-ts-docs.up.railway.app/docs/home)で表示している公式記事のMarkdownミラーです。

## 記事を編集する

1. [`content/docs`](content/docs)から対象の`.md`ファイルを開きます。
2. 先頭のfrontmatter（`title`、`description`、`path`など）を残して本文を編集します。
3. `main`へcommitすると、通常5分以内にWikiへ取り込まれます。

Wiki画面で保存した変更も、このリポジトリへ自動commitされます。同じページをGitHubとWikiで同時に編集せず、一方の反映を確認してから次を編集してください。

## 保存されるもの

このリポジトリが保存するのはページ本文です。ユーザー、権限、履歴、検索インデックス、添付ファイルはRailwayのSQLiteと`/data` Volumeにあるため、別途バックアップします。

---

This repository mirrors the Markdown shown on the [official documentation site](https://kawaii-wiki-ts-docs.up.railway.app/docs/home). Edit files under [`content/docs`](content/docs); commits to `main` are imported into the wiki on the next sync, normally within five minutes. Wiki edits are committed back here automatically. SQLite and uploaded assets still require a separate backup.
