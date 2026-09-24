---
url: https://alchemy.run/sql/drizzle/postgres
title: "Postgres"
description: "Provider-neutral Postgres schemas, Effect-native Drizzle queries, and connections for Workers, Lambda, and servers."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

`Drizzle.Postgres` provides typed, Effect-native queries for any compatible Postgres connection, whether your application runs on Workers, Lambda, or a server. Choose a database in [SQL databases](../databases.md) and supply its connection through your runtime’s binding or configuration.

Install the optional dependencies:

```sh
bun add drizzle-orm@1.0.0-rc.5-ab785fc @effect/sql-pg pg
bun add -d drizzle-kit@1.0.0-rc.5-ab785fc
```

## Define the schema

Drizzle schemas are plain TypeScript modules using the `pg-core` column builders:

```typescript
import { defineRelations } from "drizzle-orm";
import { integer, pgTable, serial, text } from "drizzle-orm/pg-core";

export const Users = pgTable("users", {
  id: serial("id").primaryKey(),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
});

export const Posts = pgTable("posts", {
  id: serial("id").primaryKey(),
  userId: integer("user_id")
    .notNull()
    .references(() => Users.id, { onDelete: "cascade" }),
  title: text("title").notNull(),
});

export const relations = defineRelations({ Users, Posts }, (t) => ({
  Users: { posts: t.many.Posts() },
  Posts: {
    user: t.one.Users({ from: t.Posts.userId, to: t.Users.id }),
  },
}));
```

## Connect

```typescript
import * as Drizzle from "alchemy/Drizzle/Postgres";
import * as Effect from "effect/Effect";
import type * as Redacted from "effect/Redacted";
import { relations, Users } from "./schema.ts";

export const makeQueries = <E, R>(
  connectionString: Effect.Effect<Redacted.Redacted<string>, E, R>,
) =>
  Effect.gen(function* () {
    const db = yield* Drizzle.Postgres(connectionString, { relations });
    return {
      listUsers: () => db.select().from(Users),
    };
  });
```

Pass an Effect that resolves a redacted URL, such as `Config.Redacted("DATABASE_URL")` or a runtime binding’s `connectionString`; wrap an already resolved redacted URL with `Effect.succeed(url)`. Call `listUsers()` inside a request or an explicit `Effect.scoped` block so the pool follows the [connection lifecycle](../effect-sql/lifecycle.md).

## Connect in a Worker

The [Cloudflare walkthrough](../../cloudflare/data/drizzle.md) supplies the URL through Hyperdrive and configures `nodejs_compat`. Database-specific setup stays in [Neon](../../neon/guides/drizzle.md) and [PlanetScale](../../planetscale/guides/drizzle.md).

## Connect in a Fly Service

The [Fly deployment guide](../../fly/data/drizzle-postgres.md) covers deployment and `ConnectPostgres`; its `connectionString` works with the same client above.

## Queries are Effects

Every builder yields directly, with `SqlError` in the typed error channel:

```typescript
import { eq } from "drizzle-orm";

const [created] = yield* db
  .insert(Users)
  .values({ name, email })
  .returning();

const [removed] = yield* db
  .delete(Users)
  .where(eq(Users.id, id))
  .returning();
```

Because `relations` was passed to `Drizzle.Postgres`, the typed `db.query.*` API is available:

```typescript
const user = yield* db.query.Users.findFirst({
  where: { id },
  with: { posts: true },
});
```

## Generate and review migrations

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/schema.ts",
  out: "./migrations",
  dialect: "postgresql",
});
```

```sh
bunx drizzle-kit generate
git add src/schema.ts drizzle.config.ts migrations
git diff --cached
git commit -m "Add database migration"
```

Generate on each schema change, stage and review the schema, SQL, and snapshots, then commit them before application. Apply the committed migrations with your existing runner before deploying code that needs the new schema. [Migrations](migrations.md) covers application ownership: [`Drizzle.Schema`](https://alchemy.run/providers/drizzle/reference#schema) is optional, and `alchemy deploy` does not replace an existing `drizzle-kit migrate` workflow.

## Where next

Choose a database and deployment guide in [SQL databases](../databases.md). For tagged-template queries without an ORM, use [Effect SQL: Postgres](../effect-sql/postgres.md).
