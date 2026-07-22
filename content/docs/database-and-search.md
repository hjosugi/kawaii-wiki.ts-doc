---
title: データベースと検索
description: SQLite、libSQL、PostgreSQL、MySQLと検索バックエンドの選び方・移行方法。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](database-and-search.en.md) | [日本語](database-and-search.md)

# データベースと検索

このページは `kawaii-wiki.ts` の `main`（確認コミット: `1774e5d`）に合わせています。最新の安定版タグは `v1.0.32` ですが、PostgreSQL、MySQL、管理画面のバックエンド表示、Elasticsearch関連の作業はその後の未リリース変更を含みます。安定版イメージだけを使う環境では、次の正式リリースまで未リリース機能を前提にしないでください。

## 利用できるデータベース

| ドライバー | 設定 | 組み込み検索 |
| --- | --- | --- |
| SQLite | `DATABASE_DRIVER=sqlite` と任意の `DATABASE_PATH` | FTS5 |
| libSQL/Turso | `DATABASE_DRIVER=libsql`、`LIBSQL_URL`、必要に応じて認証トークンとレプリカファイル | FTS5 |
| PostgreSQL | `DATABASE_DRIVER=postgres` と `DATABASE_URL` | `tsvector` |
| MySQL | `DATABASE_DRIVER=mysql` と `DATABASE_URL` | `FULLTEXT` |

PostgreSQL/MySQLでは `DATABASE_SSL=false|true|require` と `DATABASE_POOL_MAX` も指定できます。サーバーは接続確認後、起動時に対象ドライバーのスキーママイグレーションを実行します。

## Elasticsearchの現在地

`main`には設定解析、クライアント、バージョン管理されたインデックス、outbox worker、ACLを考慮した検索クエリがあります。ただし起動処理への組み込みはまだ完了していません。`SEARCH_BACKEND=elasticsearch` を設定すると、FTSへ黙ってフォールバックせず起動を停止します。本番では既定の組み込み検索を使ってください。

## SQLiteから移行する

移行先は空のPostgreSQLまたはMySQLデータベースである必要があります。最初にSQLiteと添付ファイルをバックアップし、追加のアプリケーションインスタンスを停止します。

~~~bash
# 1. 書き込みなしで件数を確認
DATABASE_DRIVER=postgres \
DATABASE_URL='postgres://user:pass@host:5432/wiki' \
bun run db:migrate-to --to postgres --from /data/ts-wiki.sqlite --dry-run

# 2. 空の移行先へコピーし、検索インデックスを再構築
DATABASE_DRIVER=postgres \
DATABASE_URL='postgres://user:pass@host:5432/wiki' \
bun run db:migrate-to --to postgres --from /data/ts-wiki.sqlite

# 3. テーブルごとの件数と内容チェックサムを照合
DATABASE_DRIVER=postgres \
DATABASE_URL='postgres://user:pass@host:5432/wiki' \
bun run db:migrate-to --to postgres --from /data/ts-wiki.sqlite --verify
~~~

MySQLへ移行する場合はドライバー、URL、`--to` を `mysql` に置き換えます。検証が成功するまで元のSQLiteと旧イメージを削除しないでください。

## 管理画面

**管理 → システム → ストレージと検索**には、稼働中のデータベースドライバー、組み込み検索エンジン、アセット保存先と状態が表示されます。同じ画面の設定生成器は環境変数の例を作りますが、稼働中のバックエンドを変更したり秘密情報を保存したりはしません。生成内容をデプロイ先のSecret/環境変数へ設定し、必要な移行とバックアップを行ってから再起動します。

R2の表示は現在、設定されたバックエンドの確認であり、実際の接続確認ではありません。
