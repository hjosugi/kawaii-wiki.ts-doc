---
title: Installation
description: Start locally with Docker only and complete the initial setup.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](installation.en.md) | [日本語](installation.md)


# Installation

## Local Startup with Docker

No need to build Bun or the source.

```bash
docker volume create kawaii-wiki-data
docker run -d --name kawaii-wiki --restart unless-stopped \
  -p 4000:4000 \
  -v kawaii-wiki-data:/data \
  -e KAWAII_WIKI_FTS_TOKENIZER=trigram \
  ghcr.io/hjosugi/kawaii-wiki.ts:1
```

Open [http://localhost:4000/setup](http://localhost:4000/setup) in your browser. On the first startup, a secure JWT secret is automatically generated at `/data/.jwt-secret` and will be retained as long as the same volume is used.

## Docker Compose

If you have already cloned the repository, you can start with just the following:

```bash
docker compose up -d
```

Stop with `docker compose down`, and view logs with `docker compose logs -f wiki`. Do not run `down -v` unless you intend to delete the data volume, as it will remove the Wiki data.

## Production Environment

In production, use HTTPS, persistent volumes, regular backups, and fixed version tags. If using external secret management, you can explicitly set `JWT_SECRET` as a random value of 32 bytes or more.

The current stable tag is `1.0.32`. See [[docs/database-and-search.en|Databases and Search]] for the distinction between stable images and unreleased features available only on source `main`.
