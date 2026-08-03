---
url: https://alchemy.run/planetscale
title: "PlanetScale"
description: "Serverless MySQL (Vitess) and Postgres with database branching — databases, branches, and credentials as Stack resources."
access_date: 2026-08-03T17:26:38.937Z
current_date: 2026-08-03T17:26:38.937Z
---

PlanetScale gives you serverless MySQL (Vitess-backed) and managed Postgres with a branch-per-PR workflow. With alchemy you declare the database, its branches, and the credentials as resources in the same Stack as your Workers — branches fork per preview stage and tear down with it.

New here? [Set up credentials](https://alchemy.run/planetscale/setup) first.

## Choose an engine

PlanetScale runs two engines, and alchemy models each as its own resource family:

- [Postgres](https://alchemy.run/planetscale/data/postgres) — managed PostgreSQL. `PostgresDatabase`, `PostgresBranch`, and `PostgresRole` (a user + password with least-privilege inherited roles).
- [MySQL](https://alchemy.run/planetscale/data/mysql) — Vitess-backed serverless MySQL. `MySQLDatabase`, `MySQLBranch`, and `MySQLPassword` (with `role: "readwriter"` instead of inherited roles).

Both families share the same shape — only the credential model differs.

## Resources

A database owns the long-lived cluster, a branch is a cheap fork, and a role materializes a user + password. In Postgres terms:

```typescript
const database = yield* Planetscale.PostgresDatabase("app-db", {
  region: { slug: "us-east" },
  clusterSize: "PS_10",
});

const branch = yield* Planetscale.PostgresBranch("app-branch", {
  database,
  isProduction: false,
});

const role = yield* Planetscale.PostgresRole("app-role", {
  database,
  branch,
  inheritedRoles: ["postgres"],
});
```

Every database and branch also accepts a `migrationsDir` of `.sql` files applied in order on each deploy — see [Migrations](https://alchemy.run/planetscale/data/migrations) for ordering, hashing, and the Drizzle pairing.

## The PlanetScale path

The role’s `origin` output (host / port / database / user / password) feeds straight into Cloudflare Hyperdrive for edge-pooled connections:

```typescript
const hyperdrive = yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
  origin: role.origin,
});
```

The guides that take this to production live under Cloudflare — each covers PlanetScale directly, Postgres and MySQL alike:

- [Hyperdrive](https://alchemy.run/cloudflare/data/hyperdrive) — provision the database, front it with Hyperdrive, and bind the connection into a Worker.
- [Add Drizzle ORM](https://alchemy.run/cloudflare/data/drizzle) — schema as a resource, migrations generated and applied on every deploy.
- [Shared database across stages](https://alchemy.run/cloudflare/data/shared-database) and [branch from a shared database](https://alchemy.run/cloudflare/data/branch-from-shared-database) — branch-per-PR preview environments off one long-lived database.

## Reference

- Postgres: [PostgresDatabase](https://alchemy.run/providers/planetscale/postgres/postgresdatabase) · [PostgresBranch](https://alchemy.run/providers/planetscale/postgres/postgresbranch) · [PostgresRole](https://alchemy.run/providers/planetscale/postgres/postgresrole) · [PostgresDefaultRole](https://alchemy.run/providers/planetscale/postgres/postgresdefaultrole)
- MySQL: [MySQLDatabase](https://alchemy.run/providers/planetscale/mysql/mysqldatabase) · [MySQLBranch](https://alchemy.run/providers/planetscale/mysql/mysqlbranch) · [MySQLPassword](https://alchemy.run/providers/planetscale/mysql/mysqlpassword)
