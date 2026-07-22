---
title: Databases and search
description: Choose database drivers, migrate from SQLite, and understand search backends.
coverPosition: center
toc: true
---
<!-- i18n: language-switcher -->
[English](database-and-search.en.md) | [日本語](database-and-search.md)

# Databases and search

This page tracks `kawaii-wiki.ts` `main` at commit `1774e5d`. The latest stable tag is `v1.0.32`, while PostgreSQL, MySQL, the Admin backend report, and Elasticsearch work include unreleased changes made after that tag. Installations using only stable images should not depend on those unreleased features until the next release.

## Available database drivers

| Driver | Configuration | Built-in search |
| --- | --- | --- |
| SQLite | `DATABASE_DRIVER=sqlite` and optional `DATABASE_PATH` | FTS5 |
| libSQL/Turso | `DATABASE_DRIVER=libsql`, `LIBSQL_URL`, and optional credentials and replica path | FTS5 |
| PostgreSQL | `DATABASE_DRIVER=postgres` and `DATABASE_URL` | `tsvector` |
| MySQL | `DATABASE_DRIVER=mysql` and `DATABASE_URL` | `FULLTEXT` |

PostgreSQL and MySQL also accept `DATABASE_SSL=false|true|require` and `DATABASE_POOL_MAX`. After checking the connection, the server runs the selected driver's schema migrations at startup.

## Elasticsearch status

`main` contains the configuration parser, client, versioned indexes, outbox worker, and ACL-safe query implementation. Application composition is not complete, however. Setting `SEARCH_BACKEND=elasticsearch` stops startup instead of silently falling back to FTS. Use the default built-in search in production.

## Migrating from SQLite

The target must be an empty PostgreSQL or MySQL database. Back up SQLite and uploaded assets first, then stop additional application instances.

~~~bash
# 1. Count rows without writing
DATABASE_DRIVER=postgres \
DATABASE_URL='postgres://user:pass@host:5432/wiki' \
bun run db:migrate-to --to postgres --from /data/ts-wiki.sqlite --dry-run

# 2. Copy into the empty target and rebuild its search index
DATABASE_DRIVER=postgres \
DATABASE_URL='postgres://user:pass@host:5432/wiki' \
bun run db:migrate-to --to postgres --from /data/ts-wiki.sqlite

# 3. Compare every table by row count and content checksum
DATABASE_DRIVER=postgres \
DATABASE_URL='postgres://user:pass@host:5432/wiki' \
bun run db:migrate-to --to postgres --from /data/ts-wiki.sqlite --verify
~~~

For MySQL, replace the driver, URL, and `--to` value with `mysql`. Keep the source SQLite database and old image until verification succeeds.

## Admin screen

**Admin → System → Storage & search** reports the active database driver, built-in search engine, asset backend, and status. Its configuration generator produces environment-variable examples; it does not switch a running backend or store secrets. Apply the generated values in the deployment environment, perform the required migration and backup, then restart.

The R2 status currently reports the configured backend and is not a live connectivity probe.
