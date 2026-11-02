---
url: https://alchemy.run/sql/drizzle/libsql
title: "libSQL"
description: "Use Drizzle with a libSQL database, such as Turso."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

`Drizzle.LibSQL` gives you typed Drizzle queries on a libSQL database, such as [Turso](../../turso.md).

Install the packages:

```sh
pnpm add @alchemy.run/drizzle@latest drizzle-orm@1.0.0-rc.5 @effect/sql-libsql
pnpm add -D drizzle-kit@1.0.0-rc.5
```

## Define the schema

```typescript
import { defineRelations } from "drizzle-orm";
import { integer, sqliteTable, text } from "drizzle-orm/sqlite-core";

export const Users = sqliteTable("users", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  email: text("email").notNull().unique(),
});

export const Posts = sqliteTable("posts", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  userId: integer("user_id")
    .notNull()
    .references(() => Users.id, { onDelete: "cascade" }),
  title: text("title").notNull(),
});

export const relations = defineRelations({ Users, Posts }, (t) => ({
  Users: { posts: t.many.Posts() },
  Posts: { user: t.one.Users({ from: t.Posts.userId, to: t.Users.id }) },
}));
```

## Make migrations

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/schema.ts",
  out: "./migrations",
  dialect: "sqlite",
});
```

```sh
bunx drizzle-kit generate
git add src/schema.ts drizzle.config.ts migrations
git commit -m "Add database migration"
```

## Create the database

```typescript
import * as Turso from "@alchemy.run/turso";
import * as Effect from "effect/Effect";

export const Db = Effect.gen(function* () {
  const group = yield* Turso.Group("Group", { location: "aws-us-east-1" });
  return yield* Turso.Database("Db", {
    group: group.name,
    migrations: "./migrations",
  });
});
```

Each deploy runs new migrations. See [Turso migrations](../../turso/migrations.md).

## Connect in a Worker

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Drizzle from "@alchemy.run/drizzle/LibSQL";
import * as Turso from "@alchemy.run/turso";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Db } from "./db.ts";
import { relations, Users } from "./schema.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const conn = yield* Turso.Connect(Db);
    const db = yield* Drizzle.LibSQL(conn, { relations });

    return {
      fetch: Effect.gen(function* () {
        const users = yield* db.select().from(Users);
        return yield* HttpServerResponse.json({ users });
      }),
    };
  }).pipe(Effect.provide(Turso.ConnectHttp)),
) {}
```

The connection opens on the first query and closes when the request ends.

## Queries are Effects

```typescript
import { eq } from "drizzle-orm";

const [created] = yield* db.insert(Users).values({ email }).returning();

const [removed] = yield* db.delete(Users).where(eq(Users.id, id)).returning();
```

With `relations`, use the `db.query.*` API:

```typescript
const user = yield* db.query.Users.findFirst({
  where: { id },
  with: { posts: true },
});
```

## Connect to any libSQL server

```typescript
const db = yield* Drizzle.LibSQL({
  url: "libsql://my-db.example.com",
  authToken: Redacted.make(token),
});
```

## Where next

[Turso migrations](../../turso/migrations.md#make-migrations-with-drizzle) shows how to make migrations from your schema. The [tutorial](../../turso/tutorial/part-1.md) uses one database per customer. [Effect SQL: libSQL](../effect-sql/libsql.md) shows plain SQL on the same connection.
