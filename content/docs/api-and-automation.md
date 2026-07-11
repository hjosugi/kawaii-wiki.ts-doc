---
title: REST API、Webhook、自動化
description: 外部連携の入口と認証。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](api-and-automation.en.md) | [日本語](api-and-automation.md)


# REST API、Webhook、自動化

## REST / OpenAPI

- 対話ドキュメント: `/api/docs`
- OpenAPI JSON: `/api/docs/openapi.json`

管理 → システム → 開発者APIから入口を開けます。APIキーは管理 → ユーザーと権限 → セキュリティで作成し、`Authorization: Bearer ...` で送ります。Web画面と同じロール・ページルールが適用されます。

GraphQLは現在未対応です。存在しない機能を暗黙に提供せず、認可・監査・レート制限をRESTと共有できる形で設計してから追加します。

## Webhook

対象URL、署名Secret、イベント種別を登録します。配信履歴には成功・失敗・再試行が表示されます。受信側ではHMAC署名を検証し、同じイベントの重複受信に耐える処理にします。

## 自動化

ページ作成・更新・削除・移動、コメントをトリガーに、ラベル・状態・パス移動・Webhook発火を構成できます。まず限定パスでテストしてから範囲を広げます。
