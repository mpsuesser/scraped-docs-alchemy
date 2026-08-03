---
url: https://alchemy.run/cloudflare/data/d1-drizzle
title: "Drizzle on D1"
description: "Manage your Drizzle schema as a resource, apply migrations to D1 on every deploy, and query from a Worker with the effect-d1 driver or the raw @effect/sql-d1 client."
access_date: 2026-08-03T19:08:21.153Z
current_date: 2026-08-03T19:08:21.153Z
---

D1 is serverless SQLite, so Drizzle needs no connection string, no Hyperdrive, and no Postgres driver — the Worker’s native D1 binding is the transport. This guide layers **Drizzle ORM** on top of [D1](d1.md): a `Drizzle.Schema` resource that regenerates migration SQL on every deploy, a `migrationsDir` wire that applies it to the database, and the `drizzle-orm/effect-d1` driver (built on `@effect/sql-d1`) for typed queries at runtime.

The same flow on Postgres (Neon or PlanetScale via Hyperdrive) is covered in [Add Drizzle ORM](drizzle.md).

## Install

`drizzle-orm`, `drizzle-kit`, and `@effect/sql-d1` are optional peer dependencies of `alchemy` — install them in your project:

```sh
bun add drizzle-orm@^1.0.0-rc.4 @effect/sql-d1
bun add -d drizzle-kit@^1.0.0-rc.4
```

## Define the schema

Drizzle schemas are plain TypeScript modules. Create `src/schema.ts` with `sqlite-core` tables:

```typescript
import { defineRelations, sql } from "drizzle-orm";
import { integer, sqliteTable, text } from "drizzle-orm/sqlite-core";

export const Users = sqliteTable("users", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .default(sql\`(unixepoch())\`),
});

export const Posts = sqliteTable("posts", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  userId: integer("user_id")
    .notNull()
    .references(() => Users.id, { onDelete: "cascade" }),
  title: text("title").notNull(),
  body: text("body").notNull(),
});

export const relations = defineRelations({ Users, Posts }, (t) => ({
  Users: {
    posts: t.many.Posts(),
  },
  Posts: {
    user: t.one.Users({
      from: t.Posts.userId,
      to: t.Users.id,
    }),
  },
}));
```

`relations` is what unlocks the typed `db.query.*` relational API later — it’s type-level wiring only and doesn’t change the generated SQL.

`Drizzle.Schema` is registered through its own `providers()` layer:

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Drizzle from "alchemy/Drizzle";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyStack",
  {
    providers: Cloudflare.providers(),
    providers: Layer.mergeAll(Cloudflare.providers(), Drizzle.providers()),
    state: Alchemy.localState(),
  },
  // ...
);
```

## Wire the schema into the database

The schema resource’s `out` output feeds the database’s `migrationsDir` input. That dependency edge orders the deploy: `Drizzle.Schema` regenerates pending migration SQL first, `Cloudflare.D1.Database` applies it second — one `alchemy deploy`, no separate `drizzle-kit generate` step in CI:

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Drizzle from "alchemy/Drizzle";
import * as Effect from "effect/Effect";

export const Database = Effect.gen(function* () {
  const schema = yield* Drizzle.Schema("app-schema", {
    schema: "./src/schema.ts",
    out: "./migrations",
    dialect: "sqlite",
  });

  return yield* Cloudflare.D1.Database("app-db", {
    migrationsDir: schema.out,
    migrationsTable: "drizzle_migrations",
  });
});
```

`dialect: "sqlite"` selects drizzle-kit’s SQLite generator. `migrationsTable` names the wrangler-compatible tracking table so already-applied migrations are skipped on subsequent deploys — [D1 migrations](d1.md#migrations) covers the mechanics, and [Drizzle migrations](../../sql/drizzle/migrations.md) the deploy-graph pattern.

## Query from the Worker with Drizzle.D1

Bind the database with `Cloudflare.D1.QueryDatabase`, hand the client to `Drizzle.D1`, and every query is an Effect you `yield*` directly:

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Drizzle from "alchemy/Drizzle";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/unstable/http/HttpServerResponse";
import { Database } from "./Db.ts";
import { relations, Users } from "./schema.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const database = yield* Database;
    const d1 = yield* Cloudflare.D1.QueryDatabase(database);
    const db = yield* Drizzle.D1(d1, { relations });

    return {
      fetch: Effect.gen(function* () {
        const users = yield* db.select().from(Users);
        return yield* HttpServerResponse.json({ users });
      }),
    };
  }).pipe(Effect.provide(Cloudflare.D1.QueryDatabaseBinding)),
) {}
```

Under the hood `Drizzle.D1` builds an `@effect/sql-d1` `D1Client` over the native binding and drives `drizzle-orm/effect-d1` with it. The client is created lazily on the first query and memoized per execution — a `fetch` / `queue` / `scheduled` event, a Durable Object call, or a Workflow run — then torn down when that execution settles. Plan- and deploy-time evaluation never touches D1.

## Relational queries

Because `relations` was passed to `Drizzle.D1`, the typed `db.query.*` namespace is available:

```typescript
const user = yield* db.query.Users.findFirst({
  where: { id },
  with: { posts: true },
});
```

The result type includes `posts: Post[]` automatically — `with` is only valid for relations you declared in `defineRelations`.

## Deploy

```sh
bun alchemy deploy
```

The first deploy generates `./migrations` from your schema, creates the D1 database, applies the `CREATE TABLE` statements, and deploys the Worker. Change the schema and deploy again — the provider diffs against the latest snapshot, writes just the delta SQL, and D1 applies the new file; an unchanged schema is a no-op.

## Prefer tagged-template SQL?

Drizzle is one consumer of the binding, not a requirement. The same `QueryDatabase` client also powers `SQL.D1` (from `alchemy/SQL/D1`) — the raw `@effect/sql-d1` client with tagged-template SQL and Effect’s generic `SqlClient` interface, no ORM involved. That path is covered in [SQL with @effect/sql-d1](d1.md#sql-with-effectsql-d1) with a complete runnable project in [`cloudflare-effect-sql-d1`](https://github.com/alchemy-run/alchemy/tree/main/examples/cloudflare-effect-sql-d1).

## Local dev

`alchemy dev` runs the same Worker against a local D1 database via miniflare — `Drizzle.D1` and `SQL.D1` work unchanged because they resolve whatever `D1Database` binding the runtime provides.

## Where next

- [Example: cloudflare-d1-drizzle](https://github.com/alchemy-run/alchemy/tree/main/examples/cloudflare-d1-drizzle) — the complete runnable project from this guide.
- [D1](d1.md) — the database resource: replication, jurisdictions, imports, cloning.
- [Drizzle migrations](../../sql/drizzle/migrations.md) — how `Drizzle.Schema` fits into the deploy graph.
- [SQL hub](../../sql.md) — the provider hub.
- [Add Drizzle ORM](drizzle.md) — the same flow on Postgres via Hyperdrive (Neon / PlanetScale).

Reference:

- [Schema](https://alchemy.run/providers/drizzle/schema) · [D1](https://alchemy.run/providers/drizzle/d1) · [Database](https://alchemy.run/providers/cloudflare/d1/database)
