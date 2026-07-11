---
title: Collaboration and Permissions
description: Users, groups, page rules, comments, notifications.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](collaboration.en.md) | [日本語](collaboration.md)


# Collaboration and Permissions

## Roles

- **admin**: Full administration including settings, users, permissions, and audits
- **editor**: Create and update permitted pages
- **viewer**: View permitted pages

## Groups and Page Rules

Add users to groups and set allow/deny rules using exact path matches, prefix matches, suffix matches, or regular expressions. Deny rules take precedence, and private page names are designed not to appear in lists, searches, or graphs.

## Comments and Notifications

You can comment on pages and mention users with `@name`. Check unread notifications from the notification bell and navigate to the page. Changes to pages with watch enabled are also included in notifications.

## Authentication

Supports local passwords, TOTP, passkeys, recovery codes, and OIDC. HTTPS is mandatory for public operation, with options to disable registration, require email verification, and enforce 2FA as needed.