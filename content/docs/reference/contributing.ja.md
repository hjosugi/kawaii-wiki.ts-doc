---
title: コントリビューション規約（英語原文）
description: Issue、PR、開発品質の規約。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](contributing.md) | [日本語](contributing.ja.md)


> ソース: `CONTRIBUTING.md` — このページはリポジトリの英語原文です。

# コントリビューション

kawaii-wiki.ts の改善にご協力いただきありがとうございます。大規模な製品範囲の変更については、実装前に Issue を開いてください。セキュリティ報告は `SECURITY.md` に記載されたプライベートな手順で行ってください。

開発には Bun 1.3.14 が必要です。リポジトリのルートから以下を実行してください:

```bash
bun install --frozen-lockfile
bun run lint
bun run typecheck
bun run test
bun run build
```

`bun run test:e2e` を実行する前に一度だけ `bunx playwright install chromium` を実行してください。PR は焦点を絞り、動作変更には回帰テストを追加し、既存の API 互換性を維持し、ユーザーや運用者が異なる対応を必要とする場合はドキュメントや変更履歴を更新してください。アーキテクチャや開発の詳細については `docs/HANDOFF.md` を参照してください。