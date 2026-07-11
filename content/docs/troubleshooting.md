---
title: トラブルシューティング
description: 起動、保存、検索、権限、表示崩れの確認項目。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](troubleshooting.en.md) | [日本語](troubleshooting.md)


# トラブルシューティング

## 起動しない

`/api/health`、コンテナログ、`JWT_SECRET`、`/data` の書き込み権限を確認します。Railway Volumeと非rootイメージで権限エラーになる場合は公式の `RAILWAY_RUN_UID` 設定を確認します。

## データが消えた

`/data` が永続Volumeへマウントされているか確認します。エフェメラル領域へ保存したデータは再デプロイで消えます。復旧前に現在のVolumeを複製してください。

## 日本語検索が弱い

管理 → 統計でCurrent tokenizerを確認します。既存Wikiでtrigramへ変更する前にバックアップし、検索インデックスを再構築します。

## ページが見えない

非公開ポリシー、ユーザーロール、グループ、ページルール、状態、公開予定を確認します。管理者には見えて匿名利用者には見えない場合、まずdenyルールを確認します。

## 表示が崩れる

ブラウザ倍率を100%へ戻し、カスタムCSSを一時的に無効化します。再現するURL、画面幅、スクリーンショット、ブラウザ情報をIssueへ添えてください。
