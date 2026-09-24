---
url: https://alchemy.run/sql/databases
title: "Databases"
description: "Compare database engines, connection paths, and deployment guides across Alchemy's SQL providers."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

Choose the database independently from the application runtime where supported. For example, a Cloudflare Worker or Hetzner application can both use Neon Postgres.

## Choose a database

| Database | Engine | Connection path | Setup |
| --- | --- | --- | --- |
| AWS RDS / Aurora | Postgres or MySQL | VPC connection; Data API on supported configurations | [RDS & Aurora](../aws/data/rds.md) · [Aurora + Drizzle](../aws/data/drizzle-aurora.md) |
| Aurora DSQL | Distributed SQL with Postgres wire compatibility | IAM-authenticated connection | [DSQL + Drizzle](../aws/data/drizzle-dsql.md) |
| Cloudflare D1 | SQLite | Native Worker binding | [D1 + Drizzle](../cloudflare/data/d1-drizzle.md) |
| Durable Object storage | SQLite per named object | Instance storage | [Durable Objects](../cloudflare/compute/durable-objects.md#sql-migrations) |
| Fly Managed Postgres | Postgres | Private connection from a Fly Service | [Managed Postgres](../fly/data/postgres.md) |
| Neon | Postgres | Direct or pooled URL; Hyperdrive from Workers | [Neon branches](../neon/data/branching.md) · [Connections](../neon/data/connections.md) |
| PlanetScale Postgres | Postgres | Role connection URL; Hyperdrive from Workers | [PlanetScale Postgres](../planetscale/data/postgres.md) |
| PlanetScale Vitess | MySQL | Password connection URL; Hyperdrive from Workers | [PlanetScale MySQL](../planetscale/data/mysql.md) |
| Prisma Postgres | Postgres | Database connection URL | [Prisma Postgres](../prisma/data/postgres.md) |
| Railway Postgres | Postgres | Private service URL or public TCP proxy | [Railway Postgres](../railway/data/postgres.md) |
| Railway MySQL | MySQL | Private service URL or public TCP proxy | [Railway MySQL](../railway/data/mysql.md) |

[Hyperdrive](../cloudflare/data/hyperdrive.md) pools connections to a database; it is not a database itself. The [Hetzner guide](../hetzner/data/drizzle-postgres.md) hosts the application on Hetzner and the database on Neon.

## Choose a client

| Engine | Typed schema queries | Raw SQL |
| --- | --- | --- |
| Postgres | [Drizzle.Postgres](drizzle/postgres.md) · [Prisma ORM](prisma/postgres.md) | [SQL.Postgres](effect-sql/postgres.md) |
| MySQL | [Drizzle.MySQL](drizzle/mysql.md) | [SQL.MySQL](effect-sql/mysql.md) |
| D1 | [Drizzle.D1](drizzle/d1.md) | [SQL.D1](effect-sql/d1.md) |
| Durable Objects | [Drizzle.DurableObject](drizzle/migrations.md#durable-object-migrations) | [Instance SQL](effect-sql/migrations.md#durable-object-migrations) |

Prisma ORM v8 requires PostgreSQL 17+. Aurora DSQL has its [own authentication and SQL constraints](../aws/data/drizzle-dsql.md); Postgres wire compatibility does not establish Prisma support or support for every PostgreSQL feature.

## Commit migration files

```sh
pnpm exec drizzle-kit generate
git add src/schema.ts drizzle.config.ts drizzle
git diff --cached
git commit -m "Add schema migration"
```

This example uses `out: "./drizzle"`; use the output directory configured by your generator and apply committed SQL through the selected database's migration workflow. Private databases require a runner with network access.

[Drizzle migrations](drizzle/migrations.md) · [Handwritten SQL](effect-sql/migrations.md) · [Prisma contracts](prisma/migrations.md) · [Runtime integration guides](../sql.md#find-an-integration)
