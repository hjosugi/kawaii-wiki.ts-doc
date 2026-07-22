---
title: 実装ハンドオフ
description: 実装状況、判断、拡張方法、注意点。
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](handoff.md) | [日本語](handoff.ja.md)


> ソース: `docs/HANDOFF.ja.md`

# kawaii-wiki.ts — ハンドオフ / 引き継ぎ資料

次にこのプロジェクトを引き継ぐ人（人間でもAIでも）向けの実践的ガイドです。ユーザー向けの概要は
[../README.md](../README.md) にあります。このドキュメントは**開発者向けハンドオフ**であり、現在の状況、なぜそうなっているのか、問題点、そして次にどこに機能を追加すればよいかを正確に示しています。

- **時点:** 2026-07-11
- **状態:** v0.5.0 — 小規模ながら*完全かつ検証済み*の垂直スライス。✅マークのついた項目はすべて実行・確認済み（テスト＋ライブHTTP＋型付きクライアント＋ビルド＋型チェック）。
- **スタック:** Bun 1.3 · Elysia · Drizzle ORM · SQLite/libSQL + FTS5 · Vue 3 · Vite ·
  UnoCSS · Pinia · CodeMirror 6 · Eden Treaty · SimpleWebAuthn（コード生成なし）。

---

## 1. 状況の概要

| 領域 | 状態 | 備考 |
|---|---|---|
| Monorepo + Bunワークスペース | ✅ | `packages/*`、`apps/*`；ルートスクリプトは `bun --filter` で制御 |
| `@kawaii-wiki/core`（純粋ドメイン） | ✅ | Result、エラー、スラッグ、権限、Markdown＋TOC/リンク抽出、レンダラープラグインの継ぎ目、バリデーション、共有の公開設定/認証プロバイダー契約 |
| DBスキーマ + FTS5マイグレーション | ✅ | SQLiteデフォルトに加えlibSQL/Turso埋め込みレプリカ対応 |
| ページサービス（CRUD） | ✅ | トランザクション処理：レンダー＋リビジョン＋FTSインデックスを一括処理 |
| 検索サービス（FTS5/BM25） | ✅ | 重み付けカラム、スニペット、プレフィックスクエリ対応 |
| ユーザー + 認証 | ✅ | 初回実行 `/setup`、ローカルパスワード、有効期限付き/取り消し可能JWT、汎用認証プロバイダー登録（OIDC実装）、TOTP＋ワンタイムリカバリーコード、パスキー、プライベートモード；初回アカウントは管理者にフォールバック |
| グループ + ページルール | ✅ | 役割デフォルトグループ、メンバーシップ、パスACLルール、拒否優先 |
| アセットアップロード | ✅ | ローカルまたはR2バイト、DBメタデータ、アップロード/ピッカーUI、ロゴ/ファビコン再利用 |
| Elysia HTTPアプリ + Eden型 | ✅ | `App`をエクスポート；エラーマッピングは集中管理 |
| Vueアプリ：表示/編集/検索/グラフ/ログイン | ✅ | パンくずリスト、ページヘッダーアクションとインサイト、ページアイコン/カバー、デスクトップツリーサイドバー＋モバイルドロワー、キーボード操作可能なグラフビュー、空状態、ランタイムブランディング |
| Markdownエディター（CodeMirror + ビジュアルモード） | ✅ | Markdownは正準形を維持；ビジュアルモードは一般的なブロックを往復可能 |
| Webhook + 自動化 | ✅ | 署名付き配信、リトライ履歴、優先度/条件/アクション付きイベント自動化ルール |
| サイト設定 | ✅ | ランタイムブランディング、テーマプリセット/フォント/背景、ナビ設定、日次ノートパス、サイトポリシー、デフォルトロケール/タイムゾーン/日付フォーマット、Webhookリトライポリシー、共有の `PublicSettings` 形状 |
| テスト / 型チェック / ビルド | ✅ | core/server Bunテスト、サービス直接カバレッジ、Web Vitestテスト；3パッケージすべて型チェック済み；Webビルド |
| モバイルシェル | ✅ | コンパクトヘッダー、タッチで見えるコマンドパレットトリガー、フォーカスマネージドナビゲーションドロワー、`xl`以下で折りたたみ可能なページTOC、全高モバイルエディターの書き込み/プレビューペイン |
| アクセシビリティシェル | ✅ | コンテンツスキップリンク、ナビ後のフォーカスされたメインランドマーク、可視フォーカスリング、ラベル付きコントロール、動作軽減対応、スケルトンローディング状態、ダイアログセマンティクス、Escape処理、アプリモーダルのフォーカストラップ/復元 |
| ルーターの認証ルートガード | ✅ | グローバルルーターガードで編集者/管理者ルートを制御 |

### リリースバッチで検証済み（証拠）
- `bun run test` は core/server BunテストとWeb Vitestコンポーネント/コンポーザブルテストを実行。
- 専用サービステストはパスキー成功/リプレイパス、認可ポリシー組み立て、検索クエリ構築、設定バリデーション、コメント、アセット、ユーザー、分析、Webhook配信リトライ状態をカバー。
- ライブAPIスモークテストは登録/ログイン、権限失敗、パス正規化、保存時レンダー、検索、更新時再インデックス、移動、削除、SSE、WebSocket認証、アセットをカバー。
- Eden Treatyクライアントはすべてのリクエスト形状（`get`/`post`/`put`/`delete`＋クエリ/ボディ）を実際のサーバー `App` 型と照合。
- `bun run typecheck`、`bun run build`、Dockerビルド、Dockerスモークはリリースチェックの一部。

---

## 2. アーキテクチャの決定（理由）

ユーザーと明示的に合意した3つの選択：

1. **SQLite + FTS5**（Postgresではない）。ゼロセットアップで高速、BM25＋重み付けカラムは「十分にリッチ」で開始可能。Drizzleは後でPostgresが必要になった場合にスキーマを移植可能に保つ。
2. **Elysia + Eden Treaty**（GraphQLではない）。ルート定義が契約そのものであり、型は**コード生成なし**でVueアプリに届く。Apolloより軽量かつ高速にビルド可能。
3. **Lean Vue**（Quasarではない）。Wiki.jsのVueから*ロジック*を移植（レンダラー、ブロックのアイデア、ストア）、しかし新鮮でモダンなUnoCSSデザイン。バンドルは小さく、デザイン制御は完全。

横断的な原則（ユーザーが求めた「FP志向アーキテクチャ」）：

- **依存は内向きに向ける。** `@kawaii-wiki/core` はI/Oもグローバルも持たない。`apps/web` と `apps/server` はcoreに依存し、互いには依存しない。ただしサーバーの*型*（Eden）は例外。
- **純粋なcore、効果は端に。** レンダリング/スラッグ/バリデーション/権限は純粋関数。
- **例外投げではなく `Result<T, E>` を使う。** サービスは結果を返す。`apps/server/src/http/errors.ts`（`unwrap`＋`onError`）が唯一のエラー→HTTP境界。
- **DI、シングルトンではない。** `createDb()` → `createServices(db)` → `createApp({ db, env })`。Wiki.jsのグローバルで可変な `WIKI` オブジェクトへの意図的な対抗策。テストは `:memory:` DBを注入。
- **原子保存。** ページ書き込みはMarkdownレンダー、履歴スナップショット、検索インデックス更新を1トランザクション内で行う（`apps/server/src/services/pages.ts`）。呼び出しが返る時点でページはレンダー済みかつ検索可能。

---

## 3. 拡張時に守るべき慣習

> これらを守ることでコードベースの一貫性を保つ。

- **純粋な新ドメインロジックは `@kawaii-wiki/core` に入れ、単体テストを書く。** DBやネットワークに触れるなら*サービス*であってcoreではない。
- **サービスはファクトリー：** `createXService(db) => { ...methods }`。想定される失敗（バリデーション/権限/競合/未発見）には `Result<T, AppError>` を返す。例外は投げない。
- **複数行の書き込みはすべて `db.transaction(...)` に入れる。** 書き込みが検索に影響する場合は同一トランザクションで `pages_fts` を更新（`pages.ts` の `reindex()` を参照）。
- **権限：** `packages/core/src/permissions.ts` の `Action` とロール→アクションマトリクスに追加し、サービスメソッドの先頭で `can(principal, action)` を呼ぶ。チェックを散らさない。
- **HTTPハンドラーは薄く保つ：** Elysiaの `t.*` でバリデートし、サービスを呼び、Resultを `unwrap()` し、プレーンなデータを返す。新しいエラー種は `packages/core/src/errors.ts` と `httpStatus()` に追加。
- **ページパスはクエリパラメータ**（`/api/page?path=...`）であり、ルートセグメントではない。wikiパスはスラッシュを含むため。Edenクライアントもこれに合わせている。
- **WebクライアントはAPI形状を `apps/web/src/lib/api.ts` に集約。** 共有契約（`PublicSettings`、`PublicAuthProvider`）は `@kawaii-wiki/core` 由来。新メソッドはこのファイルに追加し、コンポーネントやストアは直接 `treaty` を呼ばない。

---

## 4. 既に遭遇した落とし穴（再発見しないように）

1. **Edenの `delete` は `(body, options)` を取る** — クエリは*第2引数*：`client().api.page.delete(null, { query: { path } })`。`delete({ query })` は静かに422になる。
2. **Eden + グローバル `onError` はエラーボディを各ルートの成功型にユニオンする。** そのため `res.data.page` は狭められない。`api.ts` で呼び出しごとに成功形状を `call<T>()` で明示し局所化。*リクエスト*（path/query/body）は完全に型チェックされているのが重要。
3. **ルートの `bun test` は無関係なテストも拾う。** ルートの `test` スクリプトは `bun test packages apps/server && bun --filter '@kawaii-wiki/web' test` にスコープされている。WebのSFCテストはVitestで行い、ルートスクリプトを単純な `bun test` に戻さない。
4. **自動説明は更新時に再生成が必要。** 古い自動要約を持ち越すと検索インデックスに古い語句が残る。`pages.update` は `description: patch.description`（undefinedなら新コンテンツから再要約）を渡す。コメント参照。
5. **`author_id` はソフトリファレンスでありFKではない。** トークンは毎リクエストでユーザーロウと再検証されるが、過去のページ/リビジョンはユーザー削除後も残る必要がある。単なるカラム（`schema.ts`/`migrate.ts`に記述）。
6. **スラッグは許可リスト方式**（`[^\p{L}\p{N}]+ → -`）で、日本語/Unicodeを保持し任意の句読点を均一に処理。ASCIIに「簡略化」しない。
7. **FTS5トークナイザーとCJK。** デフォルトの `unicode61` は日本語を分割しない。CJK多用コンテンツは最初のマイグレーション前に `KAWAII_WIKI_FTS_TOKENIZER=trigram` を設定。既存DBはバックアップ後に `KAWAII_WIKI_FTS_TOKENIZER=trigram bun run db:reindex-search` を実行。
8. **drizzle-kitは使わない。** DDL（FTS5仮想テーブル含む）は `migrate.ts` に手書きで、`schema.ts` と同期を保つ必要あり。後でdrizzle-kitを採用してもFTS5は手動マイグレーションが必要。
9. **バックアップはSQLite優先。** `data/ts-wiki.sqlite` は `.backup` を使い、`data/assets/` はコピー。Gitミラーはコンテンツミラーであり完全なシステムバックアップではない。
10. **構造化ログは標準出力JSON＋オプションのDB監査行。** リクエストログはメソッド/パス/ステータス/時間/IP/ユーザーをカバー。監査ログは認証、ページ/管理者の変更、アセットアップロード、Git同期、コラボ自動保存をカバー。`KAWAII_WIKI_AUDIT_DB=false` で標準出力のみモード。
11. **リアルタイム認証分離。** SSEとYjsコラボはトークン必須。パブリックモードではプレゼンスは装飾的だが、プライベートモードではソケットオープン前に有効トークンが必要。
12. **カスタムhead HTMLは二重に制御。** サーバーは `KAWAII_WIKI_ALLOW_HEAD_INJECTION=true` でない限り保存済み `customHeadHtml` を抑制。Webアプリは信頼済み `<script>` タグを再生成し、解析スニペットを実行可能にする。これは管理者信頼コードとして扱い、ユーザーコンテンツではない。
13. **サイトの日付設定は両レンダラーに供給。** 管理者設定と `KAWAII_WIKI_DEFAULT_LOCALE` / `KAWAII_WIKI_TIMEZONE` / `KAWAII_WIKI_DATE_FORMAT` はサーバーの保存時レンダーとブラウザプレビューに影響。新しい表示デフォルト追加時は `markdownEnhance.ts`、`i18n.ts`、`createServices()` を同期。

---

## 5. ロードマップ — 次のステップ（優先順）

2026-06-14以降のプロダクト方針：**ストレージ/検索の幅よりも日常の使いやすさと再利用可能なUIコンポーネントを優先。** SQLite + FTS5と現在のストレージモデルはシンプルに保ち、wikiの閲覧、作成、編集、再編成、ミスからの回復が優れた体験になるまで。カレンダー/イベントワークフローは優先的な表面：会議メモ、プロジェクトイベント、ローンチ日、締切、埋め込みスケジュールはページに簡単に貼り付けられ、実際のカレンダーに簡単に送信できるべき。

参考にすべきパターン：
- **BookStack**（`https://www.bookstackapp.com/`）：ページリビジョン、画像管理、シンプルなコンテンツ整理がコアUX。
- **Outline**（`https://www.getoutline.com/`）：高速なドキュメント/コレクションワークフロー、迅速な作成、ドキュメント単位の共有がバックエンドの多様性より重要。
- **Docusaurus**（`https://docusaurus.io/docs/sidebar`）：生成されたサイドバー/カテゴリと予測可能なドキュメントナビゲーションで大規模ドキュメントを手動管理なしにナビゲート可能に。
- **Wiki.js**（`https://docs.requarks.io/`）：構造化されたページツリーナビゲーション、アセット、エディター、ページ管理がユーザーが日常的に触れる実用的な表面。

各項目は**どこにプラグインするか**を示す。

**高価値・低労力**
- [x] **貼り付け可能なカレンダーイベントカード** — Markdownフェンスに `event` 情報文字列を付けると `wiki-event-card` としてレンダー。タイトル、時間、タイムゾーン、場所、URL、説明、Googleカレンダーテンプレートリンク、ダウンロード可能な `.ics` を含む。`MarkdownEditor.vue` に `Event` スニペットボタンあり。OAuth不要の最初のカレンダー垂直スライス。
- [x] **グラフビュー** — coreの `extractPageLinks()` は `[[Wiki Links]]` と内部Markdownリンクを処理。`pages.graph()` はページ/欠損ノード＋エッジを返す。`GET /api/graph`。再利用可能な `InteractiveGraph.vue` はObsidian風のフォースグラフ（ズーム、パン、ノードドラッグ、ローカル/グローバルモード、深度、欠損ノード切替、リンク度によるノードサイズ）。`PageView.vue` は右レールにコンパクトなローカルグラフ、`GraphView.vue` は全体グラフを表示。
- [x] **リーダークロームコンポーネント** — `PageHeader`、`WikiBreadcrumbs`、`PageTree`、`EmptyState` を追加。ページビューはコピー・パス、編集、新規子ページ、更新日時メタデータ、API変更なしの構造化サイドバーを持つ。
- [x] **ページ名変更 / 移動** — `pages.ts` の `move(oldPath, newPath, principal)`；`POST /api/page/move`；`Api.movePage()`；`PageEdit.vue` で編集可能なパス；テストはパス正規化、FTS保持、競合拒否をカバー。
- [x] **ページ履歴UI** — `pages.history()`、`/api/page/history`、`Api.history()`、`HistoryView.vue` でリビジョン閲覧、差分表示、リビジョン復元を提供。
- [x] **アセット画像UI** — `MarkdownEditor.vue` はアップロードボタン、ドラッグ＆ドロップ、画像貼り付けアップロード、既存アセット閲覧用 `AssetPicker.vue` をサポート。
- [x] **エディターの使いやすさ** — ツールバーボタンは見出し/太字/リンク/コード/表/イベント/アセットをカバー。画像貼り付け対応、`.ics` インポート、未保存変更警告、保存状態を `MarkdownEditor.vue` / `PageEdit.vue` に表示。
- [x] **カレンダーのインポート/エクスポートUX** — `.ics` パースは `@kawaii-wiki/core` にあり、エディターは `.ics` イベントを `event` フェンスにインポート可能。レンダーされたイベントカードはダウンロード可能な `.ics` をエクスポート。
- [x] **クイックスイッチャー / コマンドパレット** — `CommandPalette.vue` はキーボード優先検索、ページジャンプ、新規ページ作成、共通ナビゲーションアクションをサポート。
- [x] **テンプレート / スターターページ** — `_new` は空白、意思決定、ハウツー、会議メモ、仕様、永続カスタムテンプレートを事前入力可能。エディターは `/_templates` でテンプレート管理可能。管理者は管理画面でも管理可能。`PageEdit.vue` は現在ページを再利用可能テンプレートとして保存可能。会議メモはブラウザのタイムゾーンを使用。
- [x] **グローバルルーター認証ガード** — `router.beforeEach` は管理者/編集ルートを制御し、ログイン用リダイレクトクエリを保持。
- [x] **ランタイムブランディングとテーマレイヤー** — 管理画面の外観設定でサイトタイトル、アクセントカラー、ライト/ダーク/システムデフォルト、ロゴ、ファビコン、フッターテキスト/リンク、カスタムCSS、制御されたカスタムhead HTMLを設定可能。CSS変数（`--c-bg`、`--c-surface`、`--c-text`、`--c-border`、`--c-accent`、`--radius`）がアプリクロームとレンダーブロックを駆動。
- [x] **設定コントロール** — 安全なサイトポリシー設定（登録、プライベートwikiモード、メール/2FA必須、セッション寿命、アップロード制限）は環境変数から初期化され、後に管理画面で管理可能。複数OIDCプロバイダーは `KAWAII_WIKI_OIDC_PROVIDERS` JSONまたは番号付き `OIDC_1_*` プレフィックスで提供可能。サイトのロケール/タイムゾーン/日付フォーマットのデフォルトはページロケールとレンダリング日付に影響。Webhookのリトライ試行/バックオフ/ボディ/エラー制限は環境変数で設定可能。
- [x] **自動化拡張** — ルールはトリガー/条件/アクションを使い、ページ作成/更新/削除/移動とコメント作成トリガー、パス/ラベル/ステータス/作成者/ロケール/スペース条件、優先度＋マッチ時停止、ラベル/ステータス/レビュー日付変更、パス下移動、カスタムWebhookイベント発火アクションをサポート。

**中規模**
- [x] **イベント抽出 + イベントインデックス** — `pages.events()` と `/api/events/index` はページ全体のイベントフェンスを抽出。`EventsView.vue` は今後/過去イベントをページリンクと `.ics` 付きで表示。
- [x] **Googleカレンダー連携（OAuthなし）** — イベントカードにGoogleカレンダーテンプレートリンクと `.ics` のインポート/エクスポート。OAuthカレンダー操作は意図的にコア外。詳細は `docs/ISSUE_RESOLUTION.md` 参照。
- [x] **バックリンク + ページビューのリンク言及** — `pages.backlinks()`、`/api/page/backlinks`、`PageView.vue` はリーダービューに直接受信リンクを表示。
- [x] **ナビゲーション管理** — ヘッダー/ルートナビは公開設定（`homePath`、組み込みナビの表示/順序、グループ化されたカスタムナビリンク）で設定可能。ページメタデータは共有サイドバーピン/手動順序をサポートし、認証ユーザーはサーバー管理のスター付きページ、折りたたみフォルダ、個人サイドバー順序を持つ。
- [x] **Markdownプラグイン / 型付きブロック** — `packages/core/src/markdown.ts` は安全なコールアウト、埋め込み、イベント、インフォボックス/プロフィール、リンク/ソーシャル、コンテンツタブ、Mermaidソースフェンスをサポート。`createRenderer({ features, plugins, fences })` と `registerFenceRenderer()` はサーバーレンダーとライブプレビューで共有。絵文字ショートコードはデフォルト有効。KaTeX数式とMermaidレンダーは管理者オプトイン。
- [x] **「ブロック」**（Wiki.jsのベストアイデア） — 現在の型付きフェンスアプローチは有用な軽量サブセットをカバーし、まだフレームワーク固有のカスタム要素は導入していない。
- [x] **ロール/権限UI + ユーザー管理** — 管理者ユーザー、ロール変更、デフォルトグループ、グループメンバーシップ管理、ページパスルールを実装済み。
- [x] **本番Webサービング** — Elysiaは `/ui` 以下に `apps/web/dist` を配信し、Dockerfileは単一の本番イメージをビルド。

**大規模 / 後回し**
- [x] OAuth/OIDC戦略 — 汎用かつ複数プロバイダーのOIDC設定、ログイン開始/コールバック、アカウント連携、登録制御、ドメイン許可リストを実装済み。
- [x] コメント — ページコメント（メンション、解決/更新/削除）、Webhookイベントを実装済み。
- [ ] タグ、多サイト、i18n、SSR。
- [ ] **Rustバックエンド検索アダプター。** SQLite FTS5はデフォルトの組み込みエンジンとして維持。`SearchIndexer` インターフェースは存在し、将来の切り替えはページ/コメント/アセット書き込み経路がFTS5詳細を知る必要なし。最良の外部オプションはRust製でタイプミス許容、検索即時UXの **Meilisearch**（`https://www.meilisearch.com/docs/getting_started/overview`）。**Tantivy**（`https://github.com/quickwit-oss/tantivy`）はより低レベルのRust/Luceneスタイルライブラリでインデクサーを自前で持ちたい場合。**Quickwit**（`https://quickwit.io/`）は大規模/ログ検索向けで、コンテンツ量が大幅に増えるまでこのwikiには過剰。SQLite FTS5トライグラム（`https://sqlite.org/fts5.html`）はCJK/部分文字列マッチングの最小アップグレード。タイプミス補正は意図的に外部アダプターに委ねる。

---

## 6. ファイルマップ（配置場所）

```
packages/core/src/
  result.ts        Result<T,E>、ok/err、map/flatMap/...        （純粋）
  errors.ts        AppErrorユニオン + httpStatus()                （純粋）
  slug.ts          normalizePath / slugifyHeading（Unicode対応）     （純粋）
  permissions.ts   Role、Action、can()                          （純粋）
  markdown.ts      createRenderer()、registerFenceRenderer()、renderMarkdown() → {html, toc}、
                   型付きフェンス、extractPageLinks()、toPlainText
  frontmatter.ts   Markdownファイルの解析/シリアライズ/パスマッピングヘルパー
  page.ts          validatePageInput()                          （純粋）
  core.test.ts     上記全ての単体テスト

apps/server/src/
  env.ts           型付き設定（loadEnv）
  db/
    schema.ts      Drizzleテーブル + 推論型
    migrate.ts     DDL（FTS5含む、`bun src/db/migrate.ts`）；トークナイザーはKAWAII_WIKI_FTS_TOKENIZER由来
    client.ts      createDb() — SQLite/libSQL + drizzle、FTS用の$clientを公開
    seed.ts        管理者＋サンプルページ（`bun run db:seed`）
    reset.ts       DBファイル削除（`bun run db:reset`）
  services/
    pages.ts       作成/更新/移動/削除/取得/一覧/グラフ — トランザクションコア
    search.ts      SearchIndexer継ぎ目 + FTS5クエリ/インデックス実装
    users.ts       カウント/検索/作成
    assets.ts      記録/一覧
    settings.ts    公開外観設定；カスタムCSSと制御されたカスタムhead HTML
    templates.ts   永続カスタムページテンプレートCRUD
    auth.ts        hashPassword/verifyPassword（Bun.password）
    index.ts       createServices(db) — コンポジションルート
  http/
    app.ts         createApp({db,env}) → Elysia；**Eden用に `App` 型をエクスポート**
    routes/templates.ts エディター制限付きページテンプレートAPI
    errors.ts      HttpError、unwrap()、toErrorResponse()
  index.ts         エントリ：env → db → app.listen
  server.test.ts   インメモリDB統合テスト（作成/検索/更新/削除/権限）

apps/web/src/
  lib/api.ts       Eden Treatyクライアント + Api.* メソッド（treatyを使う唯一の場所）
  lib/branding.ts  ランタイムタイトル/ファビコン/カスタムCSS/カスタムhead HTML適用
  lib/i18n.ts      EN/JAメッセージカタログ + 日付/時間フォーマット設定
  lib/markdownEnhance.ts  コードコピー、KaTeX CSS、Mermaidレンダー、コンテンツタブ強化
  lib/pageTemplates.ts  組み込みスターターテンプレート + 永続テンプレートヘルパー
  lib/realtime.ts  SSE/WebSocketクライアントヘルパー
  composables/     useSearch、useTheme、useMarkdownFeatures、usePresence、useForceGraph、動作軽減
  stores/          auth.ts、pages.ts（Pinia）
  router/index.ts  ルート（/_login /_search /_graph /_new /_edit/:path /:path）＋ paramToPath()
  components/      AppHeader.vue、AppFooter.vue、MarkdownEditor.vue、InteractiveGraph.vue、
                   PageHeader.vue、PageTree.vue、PageTemplatesPanel.vue、WikiBreadcrumbs.vue、
                   PageComments.vue、PageToc.vue、CommandPalette.vue、VisualEditor.vue、
                   CollabEditor.vue、AssetPicker.vue、ImageUploadDialog.vue、ModalDialog.vue、
                   DrawerSheet.vue、EmptyState.vue、Skeleton.vue、ShortcutsHelp.vue
  components/admin/ AdminStatsPanel、AdminPagesPanel、AdminAppearancePanel、AdminPolicyPanel、
                   AdminSecurityPanel、AdminUsersPanel、AdminGroupsPanel、AdminPageRulesPanel、
                   AdminWebhook*、AdminAutomationPanel、AdminAssetsPanel、AdminTrashPanel
  views/           AdminView.vue、PageView.vue、PageEdit.vue、PageTemplatesView.vue、
                   SearchView.vue、GraphView.vue、EventsView.vue、ChangesView.vue、
                   LinksView.vue、TagsView.vue、LoginView.vue、SetupView.vue、
                   SharedPageView.vue、UserProfileView.vue
  main.ts、App.vue、app.css、uno.config.ts、vite.config.ts

docs/
  KAWAII_WIKI_DESIGN_RFC.md   kawaii/pop/game-wiki向けプロダクト/デザイン方針
  INLINE_COMMENTS_RFC.md      ブロック/見出しアンカー付きコメントの決定
  LIVE_PREVIEW_EDITOR_SPIKE.md オプションのCodeMirrorライブプレビューエディタースパイク
```

新しいトップレベルモジュール、大きなコンポーネント、ドキュメントが追加されたリリース時は `rg --files packages/core/src apps/server/src apps/web/src docs` でこのマップを最新に保つ。

---

## 7. 拡張レシピ

**APIエンドポイント追加** → サービスメソッド追加（`Result`を返す）→ `http/app.ts` にルート追加（`t.*`スキーマ、`unwrap()`）、単一チェーンインスタンスを保ちEdenの `App` 型を更新 → `web/src/lib/api.ts` に `Api.*` ラッパー追加 → ストア/ビューから使用。

**新エンティティ追加** → `db/schema.ts` にテーブル → `db/migrate.ts` に対応DDL → `services/` に `createXService` → `services/index.ts` に登録 → `http/app.ts` にルート。

**権限追加** → `core/permissions.ts` の `Action` とマトリクス拡張 → サービスメソッドで `can(principal, action)`。

**Markdown機能追加** → ブロック形状なら `registerFenceRenderer('name', render)` で型付きフェンスレンダラー推奨、または `createRenderer({ plugins })` でisolated markdown-it実験/テスト。パイプラインは等価なのでサーバーの保存時レンダーとエディターのライブプレビューで同じコア動作を共有。

---

## 8. 実行方法

```bash
bun install
bun run db:seed     # admin@example.com / password  + サンプルページ
bun run dev         # サーバー :4000 + Web :5180
bun run test        # core/server Bunテスト + Web Vitestテスト
bun run typecheck   # すべてのワークスペース
bun run build       # Web本番ビルド
```

リファレンスリポジトリは `reference/`（`wiki-main` v2、`wiki-vega` v3）にありgitignoreされている — ローカル学習用のみ。
