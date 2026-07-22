---
title: デプロイ、更新、ロールバック
description: DockerとRailwayで安全に更新する手順。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](deployment-and-updates.en.md) | [日本語](deployment-and-updates.md)


# デプロイ、更新、ロールバック

## Docker

`/data` を永続Volumeへマウントします。DockerイメージはJWT secretを初回に `/data/.jwt-secret` へ生成するため、更新時も同じVolumeを使います。Secret管理基盤を使う場合は `JWT_SECRET` を明示できます。直接ポート4000を公開せず、HTTPSリバースプロキシを使います。

## Railway

Docker Imageに `ghcr.io/hjosugi/kawaii-wiki.ts:VERSION`、Volumeに `/data`、PORTに `4000` を設定します。公開ドメインと `KAWAII_WIKI_PUBLIC_ORIGIN` を一致させます。

## 更新

1. Volumeバックアップを作る
2. CHANGELOGとUPGRADINGを読む
3. イメージタグを新しい固定版へ変更
4. 再デプロイ
5. health、ログイン、検索、添付を確認

1.xのDBマイグレーションは起動時に自動適用されます。`latest` の無条件追従より、現在の安定版 `1.0.32` のような固定タグを推奨します。`main`には安定版タグより後の未リリース変更も含まれるため、ソースビルドを使う場合はコミットSHAも固定してください。

SQLiteからPostgreSQL/MySQLへ移行するときは、単に `DATABASE_DRIVER` を変更せず、[[docs/database-and-search|データベースと検索]] のdry-run・移行・検証手順を実行します。

## ロールバック

問題があれば旧イメージタグへ戻します。DBマイグレーションを含む更新では、互換性を確認し、必要なら更新前Volumeバックアップから復元します。
