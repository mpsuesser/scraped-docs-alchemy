---
url: https://alchemy.run/cloudflare/data/drizzle
title: "Add Drizzle ORM"
description: "Deploy a Cloudflare Worker with Drizzle queries, Hyperdrive connections, and reviewed, committed migrations on Neon or PlanetScale."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

This walkthrough connects a Worker to Postgres through Hyperdrive; [SQL](../../sql.md) owns the portable client APIs. For Cloudflare’s SQLite database, use [Drizzle on D1](d1-drizzle.md).

## Install the client and generator

```sh
bun add drizzle-orm@1.0.0-rc.5-ab785fc @effect/sql-pg pg
bun add -d drizzle-kit@1.0.0-rc.5-ab785fc @types/pg
```

Keep the exact Drizzle prerelease pins together for the Effect integration; `drizzle-kit` is a development dependency.

## Define the schema

```typescript
import { integer, pgTable, serial, text, timestamp } from "drizzle-orm/pg-core";

export const Users = pgTable("users", {
  id: serial("id").primaryKey(),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
  createdAt: timestamp("created_at", { withTimezone: true })
    .notNull()
    .defaultNow(),
});

export const Posts = pgTable("posts", {
  id: serial("id").primaryKey(),
  userId: integer("user_id")
    .notNull()
    .references(() => Users.id, { onDelete: "cascade" }),
  title: text("title").notNull(),
  body: text("body").notNull(),
  createdAt: timestamp("created_at", { withTimezone: true })
    .notNull()
    .defaultNow(),
});
```

This is an ordinary Postgres schema; it has no Worker-specific configuration.

## Configure migration generation

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/schema.ts",
  out: "./migrations",
  dialect: "postgresql",
});
```

The generator writes SQL and snapshots to `migrations`; it does not need database credentials.

## Generate and review migrations

```sh
bunx drizzle-kit generate
git diff -- src/schema.ts migrations
git status --short
git add src/schema.ts drizzle.config.ts migrations
git diff --cached
git commit -m "Add database migration"
```

Generate whenever the schema changes, review the schema, SQL, and snapshots together, and commit them before deployment. The optional [`Drizzle.Schema` resource](https://alchemy.run/providers/drizzle/reference#schema) is a separate integration, not part of this workflow.

## Configure the database

Use [Neon’s database configuration](../../neon/guides/drizzle.md#feed-migrations-from-the-schema) to export `NeonDb` from `src/Db.ts` with `migrations: "./migrations"`. That guide owns the project, branch, and connection options; [PlanetScale configuration](../../planetscale/guides/drizzle.md) is the alternative below.

## Add Hyperdrive

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Effect from "effect/Effect";
import { NeonDb } from "./Db.ts";

export const Hyperdrive = Effect.gen(function* () {
  const { branch } = yield* NeonDb;
  return yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
    origin: branch.origin,
    dev: branch.pooledOrigin,
  });
});
```

Hyperdrive pools the direct Neon origin; local development uses the pooled origin when it bypasses Hyperdrive.

## Open the connection with Drizzle.Postgres

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Drizzle from "alchemy/Drizzle/Postgres";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/unstable/http/HttpServerResponse";
import { Hyperdrive } from "./Hyperdrive.ts";
import { Users } from "./schema.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  {
    main: import.meta.url,
    compatibility: { flags: ["nodejs_compat"] },
  },
  Effect.gen(function* () {
    const hd = yield* Cloudflare.Hyperdrive.Connect(Hyperdrive);
    const db = yield* Drizzle.Postgres(hd.connectionString);

    return {
      fetch: Effect.gen(function* () {
        const users = yield* db.select().from(Users);
        return yield* HttpServerResponse.json(users);
      }),
    };
  }).pipe(Effect.provide(Cloudflare.Hyperdrive.ConnectBinding)),
) {}
```

`nodejs_compat` enables the Node APIs used by `pg`, and `ConnectBinding` supplies the Worker’s Hyperdrive binding. Queries yield directly as Effects and acquire their pool only within the request’s [scope](../../sql/effect-sql/lifecycle.md).

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "alchemy/Cloudflare";
import * as Neon from "alchemy/Neon";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Api from "./src/Api.ts";

export default Alchemy.Stack(
  "MyStack",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), Neon.providers()),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.url };
  }),
);
```

Only the runtime and database providers are needed; committed migration files do not require `Drizzle.providers()`.

## Deploy

```sh
bun alchemy deploy
```

The configured branch applies pending committed SQL before the Worker serves queries; deployment does not generate new migrations. Keep an existing `drizzle-kit migrate` workflow unless you deliberately choose Alchemy-managed application, as described in [migration ownership](../../sql/drizzle/migrations.md).

## Define relations for typed db.query

```typescript
import { defineRelations } from "drizzle-orm";

export const relations = defineRelations({ Users, Posts }, (t) => ({
  Users: { posts: t.many.Posts() },
  Posts: {
    user: t.one.Users({
      from: t.Posts.userId,
      to: t.Users.id,
    }),
  },
}));
```

Append the relations after the table definitions; this metadata enables the relational query API without changing migration SQL.

## Pass relations to Drizzle.Postgres

```typescript
import { Users } from "./schema.ts";
import { relations, Users } from "./schema.ts";

const db = yield* Drizzle.Postgres(hd.connectionString);
const db = yield* Drizzle.Postgres(hd.connectionString, { relations });
```

The returned client now exposes `db.query.Users` and `db.query.Posts` with typed relation names.

```typescript
const users = yield* db.select().from(Users);
return yield* HttpServerResponse.json(users);
const user = yield* db.query.Users.findFirst({
  where: { id: 1 },
  with: { posts: true },
});
return yield* HttpServerResponse.json({ user });
```

`with: { posts: true }` includes the related posts in the inferred result type.

## Iterate on the schema

```sh
bunx drizzle-kit generate
git add src/schema.ts migrations
git diff --cached
git commit -m "Update database schema"
bun alchemy deploy
```

Review each generated change before committing and deploy only those committed files. Reverting a TypeScript schema does not undo applied SQL; author and review a new migration or follow your database’s restore procedure.

## Use PlanetScale instead

```typescript
import * as Neon from "alchemy/Neon";
import * as Planetscale from "alchemy/Planetscale";

    providers: Layer.mergeAll(Cloudflare.providers(), Neon.providers()),
    providers: Layer.mergeAll(Cloudflare.providers(), Planetscale.providers()),
```

Use the [PlanetScale guide](../../planetscale/guides/drizzle.md) to export `PlanetscaleDb` with committed migrations; it owns both Postgres role and MySQL password configuration.

## Connect a PlanetScale Postgres role

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Effect from "effect/Effect";
import { PlanetscaleDb } from "./Db.ts";

export const Hyperdrive = Effect.gen(function* () {
  const { role } = yield* PlanetscaleDb;
  return yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
    origin: role.origin,
    dev: role.pooledOrigin,
    caching: { disabled: true },
  });
});
```

The Worker, Postgres schema, and queries remain unchanged.

## Connect a PlanetScale MySQL password

```typescript
// Inside the Hyperdrive effect, replacing the Postgres role configuration:
const { password } = yield* PlanetscaleDb;
return yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
  origin: password.origin,
  caching: { disabled: true },
});
```

Select the MySQL resource variant in the [PlanetScale guide](../../planetscale/guides/drizzle.md#mysql-apply-and-connect); there is no pooled password origin.

## Select the MySQL client

```sh
bun add @effect/sql-mysql2 mysql2
```

```typescript
import * as Drizzle from "alchemy/Drizzle/Postgres";
import * as Drizzle from "alchemy/Drizzle/MySQL";

const db = yield* Drizzle.Postgres(hd.connectionString, { relations });
const db = yield* Drizzle.MySQL(hd.connectionString, { relations });
```

Use the [MySQL schema and generator configuration](../../sql/drizzle/mysql.md), then generate, review, and commit MySQL migrations before deploying. The client uses [Workers-safe defaults](../../sql/effect-sql/mysql.md#workers-defaults); MySQL-specific query differences stay in the portable client guide.

## Where to from here

Use [Neon](../../neon.md) or [PlanetScale](../../planetscale.md) for database service behavior and [SQL databases](../../sql/databases.md) to discover other providers. [Branch from a shared database](branch-from-shared-database.md) covers the Worker deployment pattern for preview environments.
