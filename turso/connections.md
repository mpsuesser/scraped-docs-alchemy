---
url: https://alchemy.run/turso/connections
title: "Connections"
description: "Query a Turso database from a Worker or Lambda."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

`Turso.Connect` gives a Worker a database's URL and a token made just for that Worker. Your own Turso API token stays on your machine.

## Connect the database

```typescript
const conn = yield* Turso.Connect(Db);
```

Add `Turso.ConnectHttp` to the Worker:

```typescript
Effect.gen(function* () {
  const conn = yield* Turso.Connect(Db);
  // ...
}).pipe(Effect.provide(Turso.ConnectHttp))
```

`conn.url` and `conn.authToken` give you the URL and token while the Worker runs.

## Query with Effect SQL

```typescript
import * as SQL from "@alchemy.run/sql/SQLite";

const sql = yield* SQL.LibSQL(conn);

const users = yield* sql`SELECT * FROM users`;
```

See [Effect SQL: libSQL](../sql/effect-sql/libsql.md).

## Query with Drizzle

```typescript
import * as Drizzle from "@alchemy.run/drizzle/LibSQL";

const db = yield* Drizzle.LibSQL(conn);

const users = yield* db.select().from(Users);
```

Pass `{ relations }` to use `db.query`. See [Drizzle: libSQL](../sql/drizzle/libsql.md).

Effect SQL needs `@alchemy.run/sql` and Drizzle needs `@alchemy.run/drizzle`. Both need `@effect/sql-libsql`:

```sh
pnpm add @alchemy.run/sql @effect/sql-libsql      # Effect SQL
pnpm add @alchemy.run/drizzle @effect/sql-libsql  # Drizzle
```

## Connect from AWS Lambda

`Turso.Connect` works the same in a Lambda. Add `@libsql/client` to `build.install` so the Lambda gets the Linux version of the driver:

```typescript
export default class Api extends Lambda.Function<Api>()(
  "Api",
  {
    main: import.meta.url,
    functionUrl: true,
    build: { install: ["@libsql/client"] },
  },
  Effect.gen(function* () {
    const conn = yield* Turso.Connect(Db);
    const sql = yield* SQL.LibSQL(conn);
    // ...
  }).pipe(Effect.provide(Turso.ConnectHttp)),
) {}
```

## Connect read-only

```typescript
const conn = yield* Turso.Connect(Db, { authorization: "read-only" });
```

Writes on this connection fail.

## Use @libsql/client directly

```typescript
import { createClient } from "@libsql/client/web";
import * as Redacted from "effect/Redacted";

const client = createClient({
  url: yield* conn.url,
  authToken: Redacted.value(yield* conn.authToken),
});
```

Read `conn.url` and `conn.authToken` inside a request, not when the Worker starts.

## Connect from somewhere else

Create a [DatabaseToken](tokens.md) and give `db.url` and the token to your app.

## Where next

- [Tokens](tokens.md)
- [Database per tenant](database-per-tenant.md) — connect to databases your app creates
- [Connect reference](https://alchemy.run/providers/turso/reference/database#connect)
