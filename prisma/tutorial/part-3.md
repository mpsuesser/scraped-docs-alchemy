---
url: https://alchemy.run/prisma/tutorial/part-3
title: "Part 3: Query Postgres"
description: "Create Prisma Postgres, bind its connection to Compute, and query it with Effect SQL."
access_date: 2026-09-18T03:55:07.187Z
current_date: 2026-09-18T03:55:07.187Z
---

Continue from [Part 2](part-2.md). Add an explicit Postgres database and a read-only endpoint that returns the database’s current time. This query needs no tables or migrations.

## Install the SQL client

```sh
bun add "@effect/sql-pg@rc"
```

Alchemy’s SQL helper uses the Effect Postgres client. Match its version to the other Effect packages.

## Define the database

Append to `src/Database.ts`, adding the Effect import at the top:

```typescript
import * as Prisma from "alchemy/Prisma";
import * as Effect from "effect/Effect";

export const Postgres = Prisma.Postgres(
  "Postgres",
  Effect.gen(function* () {
    const project = yield* Project;
    return { project, region: "eu-west-3" as const, branchGitName: "main" };
  }),
);
```

The database belongs to the existing Project’s `main` branch. Keep the Project definition from Part 1, including `createDatabase: false`.

## Define a connection

```typescript
export const Connection = Prisma.Connection(
  "Connection",
  Effect.gen(function* () {
    const database = yield* Postgres;
    return { database, name: "api" };
  }),
);
```

A Connection supplies the credentials that the API will use. Its database dependency means you do not need to separately yield Postgres in the Stack.

## Bind the connection to Compute

```typescript
import { Project } from "./Database.ts";
import { Connection, Project } from "./Database.ts";

  Effect.gen(function* () {
    const db = yield* Prisma.Connect(Connection);
    return {
      fetch: Effect.gen(function* () {
        // Keep the request handler from Part 2.
      }),
    };
  }),
  }).pipe(Effect.provide(Prisma.ConnectBinding)),
```

Apply this change to the **runtime** Effect, the third argument to `Prisma.Compute`, not to the properties Effect. The binding makes the connection URL available at runtime without placing credentials in source.

## Create the SQL client

```typescript
import * as SQL from "alchemy/SQL/Postgres";

    const db = yield* Prisma.Connect(Connection);
    const sql = yield* SQL.Postgres({ url: db.databaseUrl });
```

The helper resolves the redacted URL and opens a connection lazily when a request executes a query. The request scope closes its pool afterward. Do not return the database URL from an HTTP handler or Stack output.

## Query the database

Insert the new route after `/api/health` and before the final 404 response:

```typescript
if (request.url === "/api/time") {
  const rows = yield* sql<{ time: string }>\`SELECT current_timestamp::text AS time\`;
  return yield* HttpServerResponse.json(rows[0]);
}
return HttpServerResponse.text("Not found", { status: 404 });
```

The timestamp comes from Postgres, not the API process.

## Handle database failures

Attach a typed error handler to the `fetch` Effect:

```typescript
return HttpServerResponse.text("Not found", { status: 404 });
}),
}).pipe(
  Effect.catchTag("SqlError", () =>
    Effect.succeed(HttpServerResponse.text("Database unavailable", {
      status: 503,
    })),
  ),
),
```

A failed SQL query returns HTTP 503 without exposing connection details. Handling `SqlError` also satisfies the HTTP handler’s typed error contract.

## Deploy the database and API

```sh
bun alchemy deploy
```

Alchemy creates the database and connection before deploying the API with its binding. Existing resources retain their logical IDs.

## Read the database time

```sh
curl https://YOUR_API_HOST/api/time
```

Expect a JSON object with a `time` string. `/api/health` continues to work without querying Postgres, and `/missing` still returns HTTP 404.

## Next

[Part 4: A Vite Frontend](part-4.md) adds a browser interface, local frontend development, and cleanup.
