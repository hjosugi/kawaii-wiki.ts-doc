---
title: インストール
description: Dockerだけでローカル起動し、初期設定を完了する。
coverPosition: center
toc: true
---

# インストール

## Dockerでローカル起動

Bunやソースのビルドは不要です。

```bash
docker volume create kawaii-wiki-data
docker run -d --name kawaii-wiki --restart unless-stopped \
  -p 4000:4000 \
  -v kawaii-wiki-data:/data \
  -e KAWAII_WIKI_FTS_TOKENIZER=trigram \
  ghcr.io/hjosugi/kawaii-wiki.ts:1
```

ブラウザで [http://localhost:4000/setup](http://localhost:4000/setup) を開きます。初回起動時に安全なJWT secretが `/data/.jwt-secret` へ自動生成され、同じVolumeを使う限り保持されます。

## Docker Compose

リポジトリを取得済みなら、次だけで起動できます。

```bash
docker compose up -d
```

停止は `docker compose down`、ログは `docker compose logs -f wiki` です。`down -v` はデータVolumeも削除するため、Wikiを消す意図がない限り実行しないでください。

## 本番環境

本番ではHTTPS、永続Volume、定期バックアップ、固定バージョンタグを使います。外部Secret管理を利用する場合は `JWT_SECRET` を32バイト以上のランダム値として明示できます。
