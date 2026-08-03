---
url: https://alchemy.run/sql/effect-sql/migrations
title: "Migrations"
description: "Commit a directory of ordered .sql files and point a database resource's migrationsDir at it — pending files apply as part of every deploy."
access_date: 2026-08-03T18:54:18.847Z
current_date: 2026-08-03T18:54:18.847Z
---

Migrations without an ORM: commit `.sql` files to a directory and
wire it into the database resource with `migrationsDir`. Each deploy
applies the pending files — there is nothing else to run.

## Write the migrations

```sql
-- migrations/0001_init.sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE
);
```

```sql
-- migrations/0002_posts.sql
CREATE TABLE posts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL REFERENCES users(id),
  title TEXT NOT NULL
);
```

## Wire the directory in

```typescript
const db = yield* Cloudflare.D1.Database("app-db", {
  migrationsDir: "./migrations",
});
```

The same prop exists on `Neon.Branch` and
`Planetscale.PostgresBranch`.

## How files are applied

- The directory is scanned recursively for `.sql` files.
- Files are ordered by numeric prefix (`0001_init.sql`,
  `0002_posts.sql`), then by name.
- Each file is content-hashed and applied files are recorded, so a
  deploy only runs what's new — an unchanged directory is a noop.
- Application is ordered and stops on the first failure; the failed
  file is retried on the next deploy.

Each database documents its own mechanics — tracking table,
transactionality, and quirks:

| Database | Mechanics |
| --- | --- |
| Cloudflare D1 | [D1 migrations](../../cloudflare/data/d1.md#migrations) — wrangler-compatible tracking table, `migrationsTable` for drizzle compatibility |
| Neon | [Neon migrations](../../neon/data/migrations.md) — applied transactionally on the branch |
| PlanetScale | [PlanetScale migrations](../../planetscale/data/migrations.md) — the same contract on Postgres and MySQL branches |

## Generated migrations target the same contract

[`Drizzle.Schema`](../drizzle/migrations.md) emits into the same
shape: its `out` output is a migrations directory, so
`migrationsDir: schema.out` orders generation before application in
one deploy. Any tool that writes ordered `.sql` files into a
directory already works.

## Where next

- [Postgres](postgres.md) /
  [D1](d1.md) — query the migrated database.
- [Drizzle migrations](../drizzle/migrations.md) — generate the
  files from a schema module instead of writing them by hand.
- [Branch from a shared database](../../cloudflare/data/branch-from-shared-database.md)
  — per-stage databases built on the same contract.
