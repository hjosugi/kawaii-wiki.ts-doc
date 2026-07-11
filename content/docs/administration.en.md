---
title: Management and Secure Operation
description: How to find management settings, appearance, policies, and auditing.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](administration.en.md) | [日本語](administration.md)


# Management and Secure Operation

The management screen features a categorized side navigation. You can search for appearance, users, Git, Webhooks, and more by name or related terms using the "Search settings" box at the top.

## Initial Settings to Configure

1. **General → Appearance**: Wiki name, logo, theme, language, time zone
2. **General → Policies**: Privacy, registration, email verification, 2FA
3. **Users and Permissions**: Users, groups, page rules
4. **System → Audit**: Administrative actions and critical events

## Secrets

JWT secret, SMTP, OIDC secret, R2 keys, and Git keys should not be saved in the wiki content or regular settings. Manage them via secret environment variables on the deployment target. Do not paste them into logs or screenshots.

## Backup

Git is a content mirror and not a complete backup. Please regularly back up the `/data` volume, which includes SQLite and attachments.