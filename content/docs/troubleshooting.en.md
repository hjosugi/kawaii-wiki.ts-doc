---
title: Troubleshooting
description: Checklist for startup, saving, searching, permissions, and display issues.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](troubleshooting.en.md) | [日本語](troubleshooting.md)


# Troubleshooting

## Won't Start

Check `/api/health`, container logs, `JWT_SECRET`, and write permissions for `/data`. If you encounter permission errors with Railway Volume and non-root images, verify the official `RAILWAY_RUN_UID` setting.

## Data Lost

Confirm that `/data` is mounted to a persistent volume. Data saved to ephemeral storage will be lost upon redeployment. Before recovery, duplicate the current volume.

## Weak Japanese Search

Check the Current tokenizer under Admin → Statistics. Before switching an existing wiki to trigram, back up your data and rebuild the search index.

## Page Not Visible

Check private policies, user roles, groups, page rules, status, and scheduled publication. If administrators can see the page but anonymous users cannot, first verify any deny rules.

## Display Issues

Reset browser zoom to 100% and temporarily disable custom CSS. Please include the URL where the issue occurs, screen width, a screenshot, and browser information when submitting an issue.