---
title: Gitミラーとバックアップ
description: Git同期の設定とデータ保護。
coverPosition: center
toc: true
---

# Gitミラーとバックアップ

## Gitミラー

管理 → システム → Gitで接続用環境変数を生成できます。ページ本文は `content/*.md` としてコミットされ、Wikiの変更はGitへ、外部コミットは明示的な同期でWikiへ取り込まれます。

アクセストークンをリポジトリURLへ埋め込まないでください。公開リポジトリの読み込み以外は、ホスト側のSSH Deploy Keyを利用します。

## 完全バックアップ

完全な復旧には次が必要です。

- SQLite DB（WAL実行中はオンラインbackup APIまたは停止後コピー）
- `/data/assets` の添付ファイル
- JWT、SMTP、OIDC、R2などのSecret
- 現在のコンテナイメージタグ

RailwayではVolumeのDaily/Weekly/Monthlyバックアップを設定できます。復元手順も定期的にテストしてください。
