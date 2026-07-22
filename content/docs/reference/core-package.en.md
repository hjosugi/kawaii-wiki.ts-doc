---
title: Core Package
description: Pure domain logic and types.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](core-package.en.md) | [日本語](core-package.md)


> Source: `packages/core/README.md`

# kawaii-wiki.ts Core

A pure TypeScript domain package shared between the server and web app.

## What You Can Learn from This Package

- Architecture separating a functional core from an imperative shell
- Representing success and failure with typed `Result<T, E>` instead of throwing exceptions from domain logic
- Input validation and normalization before persistence
- Permission checks expressed as small policy tables
- Markdown rendering and slug generation independent of HTTP and databases
- Extension points for the Markdown renderer supporting plugins, feature flags, and typed fence blocks

## How to Run

Run from the root of the repository:

```bash
bun test packages/core
bun --filter '@kawaii-wiki/core' typecheck
```

## Files to Read First

| File | Role |
| --- | --- |
| `src/result.ts` | Success and failure values used throughout the app |
| `src/errors.ts` | Typed application errors |
| `src/page.ts` | Validation of page input |
| `src/permissions.ts` | Centralized authorization policies |
| `src/markdown.ts` | Markdown to HTML conversion, feature flags, typed fences, `createRenderer()`, `registerFenceRenderer()` |
| `src/slug.ts` | Safe handling of Unicode slugs |

## Exercises

1. Add a new permission action and ensure its behavior with tests.
2. Add validation for a new page field without importing server code.
3. Extend Markdown rendering with `createRenderer({ features, plugins, fences })` while maintaining deterministic output.
4. Add a failing test first, then update the pure function.
