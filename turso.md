---
url: https://alchemy.run/turso
title: "Turso"
description: "Create Turso SQLite databases and query them from your Workers."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

Turso hosts SQLite databases for you. Create them in the same Stack as your Workers, and query them with Effect SQL or Drizzle.

New here? [Set up credentials](turso/setup.md) first, then [build the tutorial](turso/tutorial/part-1.md).

## Create a database

Every database lives in a group. The group decides where its databases run.

```typescript
import * as Turso from "@alchemy.run/turso";

export const Group = Turso.Group("Group", { location: "aws-us-east-1" });

export const Db = Effect.gen(function* () {
  const group = yield* Group;
  return yield* Turso.Database("Db", {
    group: group.name,
    migrations: "./migrations",
  });
});
```

Each deploy runs any new migrations. See [Migrations](turso/migrations.md).

## Query it from a Worker

`Turso.Connect` gives the Worker the database’s URL and a token. Pass it to [Drizzle](sql/drizzle/libsql.md) or [Effect SQL](sql/effect-sql/libsql.md):

```typescript
const conn = yield* Turso.Connect(Db);
const db = yield* Drizzle.LibSQL(conn);
```

See [Connections](turso/connections.md).

## One database per tenant

Turso databases are cheap, so you can give each customer their own. `Turso.ManageDatabases` lets a Worker create, query, and delete databases in one group:

```typescript
const tenants = yield* Turso.ManageDatabases(Tenants);
const acme = yield* tenants.create("acme");
const sql = yield* SQL.LibSQL(tenants.connect(acme.name));
```

The [tutorial](turso/tutorial/part-1.md) builds this step by step. The [Database per tenant](turso/database-per-tenant.md) guide has the details.

## Where next

- [Groups](turso/groups.md) and [Databases](turso/databases.md)
- [Tokens](turso/tokens.md) for apps outside your Stack
- [Preview databases](turso/preview-databases.md), one copy per pull request
- [Team & organization](turso/organization.md): invites, members, and settings
