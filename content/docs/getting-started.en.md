---
title: Getting Started
description: From initial setup to creating your first page.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](getting-started.en.md) | [日本語](getting-started.md)


# Getting Started

## Initial Setup

On first access, `/setup` will open. Set the wiki name, administrator name, email, strong password, theme, and search method. If you mainly use Japanese, set the search method to **trigram**.

The wiki name is prominently displayed in the header, browser title, and homepage. You can change it later under **Admin → General → Appearance**.

## Things to Check First

- You can log in as an administrator
- You can edit and save the homepage
- Edited content appears in search results
- Attached images remain after a restart
- `/api/health` returns `ok: true`

## Your First Page

Click "New Page" and enter a title. The path is automatically generated from the title. Choose a template or start from a blank draft, and before saving, review the status, labels, and scheduled publication.

Pages can be linked with `[[page path]]`. Links to non-existent pages appear as uncreated pages in the graph and broken link lists.