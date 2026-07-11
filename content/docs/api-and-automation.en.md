---
title: REST API, Webhook, Automation
description: Entry points and authentication for external integration.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](api-and-automation.en.md) | [日本語](api-and-automation.md)


# REST API, Webhook, Automation

## REST / OpenAPI

- Interactive documentation: `/api/docs`
- OpenAPI JSON: `/api/docs/openapi.json`

You can access the entry point from Admin → System → Developer API. Create an API key under Admin → Users and Permissions → Security, and send it with `Authorization: Bearer ...`. The same roles and page rules as the web interface apply.

GraphQL is currently not supported. We will add it only after designing it to share authorization, auditing, and rate limiting with REST, without implicitly providing unsupported features.

## Webhook

Register the target URL, signature secret, and event types. Delivery history shows success, failure, and retries. On the receiving side, verify the HMAC signature and implement processing that tolerates duplicate reception of the same event.

## Automation

You can configure label changes, status updates, path moves, and webhook triggers based on page creation, update, deletion, move, or comments. Test first with a limited path before expanding the scope.