---
title: Git Mirror and Backup
description: Git synchronization settings and data protection.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](git-and-backups.en.md) | [日本語](git-and-backups.md)


# Git Mirror and Backup

## Git Mirror

You can generate environment variables for connection under Admin → System → Git. The page content is committed as `content/*.md`, changes in the Wiki are pushed to Git, and external commits are incorporated into the Wiki through explicit synchronization.

Do not embed access tokens in the repository URL. Except for reading public repositories, use the host-side SSH Deploy Key.

## Full Backup

The following are required for complete recovery:

- SQLite DB (use the online backup API during WAL operation or copy after stopping)
- Attachments in `/data/assets`
- Secrets such as JWT, SMTP, OIDC, R2, etc.
- The current container image tag

On Railway, you can configure daily/weekly/monthly backups for volumes. Please regularly test the restoration procedures as well.