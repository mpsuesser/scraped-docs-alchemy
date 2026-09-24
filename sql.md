---
url: https://alchemy.run/sql
title: "SQL"
description: "Choose a database, connect with Effect SQL, Drizzle, or Prisma ORM, and deploy committed migrations."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

## Choose a database

See [Databases](sql/databases.md) to compare engines and connection paths, with links directly to each database’s setup guide.

## Choose a client

| Client | Guides |
| --- | --- |
| Effect SQL | [Postgres](sql/effect-sql/postgres.md) · [MySQL](sql/effect-sql/mysql.md) · [D1](sql/effect-sql/d1.md) · [Durable Objects](sql/effect-sql/migrations.md#durable-object-migrations) |
| Drizzle | [Postgres](sql/drizzle/postgres.md) · [MySQL](sql/drizzle/mysql.md) · [D1](sql/drizzle/d1.md) · [Durable Objects](sql/drizzle/migrations.md#durable-object-migrations) |
| Prisma ORM v8 | [Contracts](sql/prisma/contracts.md) · [Postgres](sql/prisma/postgres.md) · [Migrations](sql/prisma/migrations.md) |

Prisma ORM v8 requires PostgreSQL 17+ and is separate from the [Prisma hosting provider](prisma.md).

## Find an integration

| Integration | Guide |
| --- | --- |
| Lambda + Aurora PostgreSQL + Drizzle | [AWS](aws/data/drizzle-aurora.md) |
| Lambda + Aurora DSQL + Drizzle | [AWS](aws/data/drizzle-dsql.md) |
| Workers + Neon or PlanetScale + Drizzle | [Cloudflare](cloudflare/data/drizzle.md) |
| Workers + D1 + Drizzle | [Cloudflare](cloudflare/data/d1-drizzle.md) |
| Durable Objects + SQLite | [Cloudflare](cloudflare/compute/durable-objects.md#sql-migrations) |
| Fly Service + Postgres + Drizzle | [Fly](fly/data/drizzle-postgres.md) |
| Hetzner Service + external Postgres + Drizzle | [Hetzner](hetzner/data/drizzle-postgres.md) |
| Prisma Compute + Postgres + Drizzle | [Prisma](prisma/data/drizzle-postgres.md) |
| Railway Service + Postgres + Drizzle | [Railway](railway/data/drizzle-postgres.md) |
| Railway Service + MySQL + Drizzle | [Railway](railway/data/drizzle-mysql.md) |
| Workers + Neon + Prisma ORM | [Cloudflare](cloudflare/data/prisma.md) |

See [runnable examples](https://github.com/alchemy-run/alchemy/blob/main/examples/README.md) for complete projects.

## Already have a database?

```typescript
import * as SQL from "alchemy/SQL/Postgres";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";

const query = Effect.gen(function* () {
  const sql = yield* SQL.Postgres({ url: Config.Redacted("DATABASE_URL") });
  return yield* sql\`SELECT 1 AS value\`;
});
```

Run queries within the application’s execution scope, or use `Effect.scoped` in a standalone program; see [connection lifecycle](sql/effect-sql/lifecycle.md).

## Manage schema changes

Generate migrations when schemas change, stage and review the schema, SQL, and snapshots with `git diff --cached`, then commit them before deployment. Apply the committed files using the selected database’s workflow.

[SQL files](sql/effect-sql/migrations.md) · [Drizzle migrations](sql/drizzle/migrations.md) · [Prisma migrations](sql/prisma/migrations.md)
