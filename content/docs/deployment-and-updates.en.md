---
title: Deploy, Update, Rollback
description: Steps for safely updating with Docker and Railway.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](deployment-and-updates.en.md) | [日本語](deployment-and-updates.md)


# Deploy, Update, Rollback

## Docker

Mount `/data` to a persistent volume. The Docker image generates the JWT secret in `/data/.jwt-secret` on the first run, so the same volume is used during updates. If you use a secret management platform, you can explicitly set `JWT_SECRET`. Do not expose port 4000 directly; use an HTTPS reverse proxy.

## Railway

Set the Docker image to `ghcr.io/hjosugi/kawaii-wiki.ts:VERSION`, mount the volume at `/data`, and set the PORT to `4000`. Make sure the public domain matches `KAWAII_WIKI_PUBLIC_ORIGIN`.

## Update

1. Create a backup of the volume
2. Read the CHANGELOG and UPGRADING files
3. Change the image tag to the new fixed version
4. Redeploy
5. Check health, login, search, and attachments

Database migrations for version 1.x are applied automatically at startup. It is recommended to use a fixed tag like `1.0.1` rather than always following `latest`.

## Rollback

If issues occur, revert to the previous image tag. For updates involving database migrations, verify compatibility and restore from the pre-update volume backup if necessary.