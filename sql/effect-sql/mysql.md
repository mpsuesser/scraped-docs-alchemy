---
url: https://alchemy.run/sql/effect-sql/mysql
title: "MySQL"
description: "SQL.MySQL turns a connection string into an @effect/sql-mysql2 client — portable tagged-template queries with typed errors and one pool per execution."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

`SQL.MySQL` provides tagged-template queries over a MySQL connection. Queries are Effects with typed `SqlError` failures, interruption, and tracing.

Install the optional MySQL driver:

```sh
bun add @effect/sql-mysql2 mysql2
```

## Connect

```typescript
import * as SQL from "alchemy/SQL/MySQL";
import * as Effect from "effect/Effect";
import type * as Redacted from "effect/Redacted";

export const makeQueries = <E, R>(
  connectionString:
    | Redacted.Redacted<string>
    | Effect.Effect<Redacted.Redacted<string>, E, R>,
) =>
  Effect.gen(function* () {
    const sql = yield* SQL.MySQL({ url: connectionString });
    return {
      listUsers: () => sql<{ id: number; name: string }>\`SELECT id, name FROM users\`,
    };
  });
```

`url` accepts a redacted URL or an Effect resolving one, such as `Config.Redacted("DATABASE_URL")` or a runtime binding’s `connectionString`. Other `@effect/sql-mysql2` options pass through. Call queries within a request or an explicit `Effect.scoped` block: the pool opens lazily and closes with that [scope](lifecycle.md).

## Workers defaults

```typescript
// Applied by default only when workerd is detected:
const workersDefaults = {
  disablePreparedStatements: true,
  poolConfig: { disableEval: true },
};
```

These defaults avoid Hyperdrive’s unsupported prepared-statement protocol and Workers’ prohibition on eval-based row parsers. Direct connections from Node or Bun retain prepared statements and mysql2’s normal parsers by default. See [Hyperdrive](../../cloudflare/data/hyperdrive.md#use-effect-sql) for Worker connection setup.

## Override driver options

`SQL.MySQL` parses URL fields and query parameters into driver configuration, including `poolConfig`. Explicit options override parsed values and detected defaults:

```typescript
const sql = yield* SQL.MySQL({
  url: connectionString,
  // For a proxy that lacks COM_STMT_PREPARE.
  disablePreparedStatements: true,
  // For a direct TLS connection.
  poolConfig: { ssl: { rejectUnauthorized: true } },
});
```

## Queries

Interpolated values are parameters, never string concatenation:

```typescript
const user = yield* sql\`SELECT * FROM users WHERE id = ${id}\`;

yield* sql\`INSERT INTO users ${sql.insert({ name, email })}\`;

const rows = yield* sql\`
  SELECT * FROM users WHERE id IN ${sql.in(ids)}
\`;
```

Rows are plain objects; supply a row type with `sql<Row>`. The [`effect/unstable/sql/Statement`](https://effect.website/) API also provides fragments, `sql.csv`, `sql.and`, and identifier escaping (backticks on MySQL).

## Errors

Failures surface as `SqlError` in the typed error channel:

```typescript
const users = yield* sql\`SELECT * FROM users\`.pipe(
  Effect.catchTag("SqlError", (e) =>
    Effect.succeed([]).pipe(Effect.tap(() => Effect.logWarning(e))),
  ),
);
```

## Transactions

Wrap a group of queries in `sql.withTransaction` — the whole effect commits or rolls back together:

```typescript
yield* sql.withTransaction(
  Effect.gen(function* () {
    yield* sql\`UPDATE accounts SET balance = balance - ${amount} WHERE id = ${from}\`;
    yield* sql\`UPDATE accounts SET balance = balance + ${amount} WHERE id = ${to}\`;
  }),
);
```

## Provide as a service

Depend on the generic `SqlClient` tag, then provide the database layer:

```typescript
import * as SqlClient from "effect/unstable/sql/SqlClient";

const makeUsers = Effect.gen(function* () {
  const sql = yield* SqlClient.SqlClient;
  return {
    find: (id: number) => sql<User>\`SELECT * FROM users WHERE id = ${id}\`,
  };
});

const users = yield* makeUsers.pipe(
  Effect.provide(SQL.MySQLLayer({ url: connectionString })),
);
```

The layer provides `SqlClient.SqlClient` and `@effect/sql-mysql2` ’s `MysqlClient` from one per-execution pool. Other drivers can satisfy the generic service, but changing engines still requires compatible SQL and transaction semantics.

## Provider setup

Choose a MySQL-compatible database and deployment guide in [SQL databases](../databases.md); the client above only needs a connection URL.

## Where next

Read [Migrations](migrations.md) to apply committed SQL files, or [Drizzle on MySQL](../drizzle/mysql.md) for typed schemas over the same driver. [Connection lifecycle](lifecycle.md) covers per-request cleanup.
