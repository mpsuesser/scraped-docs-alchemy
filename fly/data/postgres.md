---
url: https://alchemy.run/fly/data/postgres
title: "Postgres"
description: "A billed Managed Postgres cluster. Bind Fly.ConnectPostgres and query with Drizzle or SQL."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

A [`Fly.Postgres`](https://alchemy.run/providers/fly/reference/postgres#postgres) is a Managed Postgres
(MPG) cluster. It is billed. Do not wrap unmanaged `fly postgres`.

Create the cluster, pass `migrations` the same way you would on
Neon or PlanetScale, then bind
[`ConnectPostgres`](https://alchemy.run/providers/fly/reference/postgres#connectpostgres) on a
[Service](https://alchemy.run/fly/compute/services). Pass
`conn.connectionString` to `Drizzle.Postgres` or `SQL.Postgres`.

## Create a cluster

`region` is required. The cluster is regional. An
[App](https://alchemy.run/fly/compute/apps) is global. Alchemy generates a unique name
unless you pass one.

```typescript
const db = yield* Fly.Postgres("Db", {
  region: "iad",
});
```

Alchemy defaults compute to [`iad`](https://alchemy.run/fly/compute/regions). Pass
another code to put the cluster somewhere else.

:::caution[Billed]
Managed Postgres is billed. Basic is about $38 per month.
:::

:::caution[Changing `region` replaces the cluster]
A new cluster is created in the new region. The old cluster is
deleted. Data is not copied.
:::

## Connect from a Service

Yield `ConnectPostgres` in the Service's constructor. Provide
`ConnectPostgresHttp`. Pass the connection string to Drizzle or
SQL.

```typescript
import * as Drizzle from "alchemy/Drizzle/Postgres";
import * as HttpServerResponse from "effect/unstable/http/HttpServerResponse";

export default class Api extends Fly.Service<Api>()(
  "Api",
  { app: Site, main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const conn = yield* Fly.ConnectPostgres(Db);
    const db = yield* Drizzle.Postgres(conn.connectionString);
    return {
      fetch: Effect.gen(function* () {
        const rows = yield* db.execute("select 1 as ok");
        return HttpServerResponse.json({ rows });
      }),
    };
  }).pipe(Effect.provide(Fly.ConnectPostgresHttp)),
) {}
```

`connectionString` is the cluster's pooled PgBouncer URI Output.
`directConnectionString` is the direct URI. Both come from
`Fly.Postgres`, packed into the Service env — not `Config.Redacted`.

## Migrations

Pass a directory, `{ dir, table? }`, or a `Drizzle.Schema`
resource. Alchemy applies pending files on deploy over the direct
URI and records them in `__alchemy_migrations`.

```typescript
const schema = yield* Drizzle.Schema("app-schema", {
  schema: "./src/schema.ts",
  out: "./migrations",
});

const db = yield* Fly.Postgres("Db", {
  region: "iad",
  migrations: schema,
});
```

```typescript
const db = yield* Fly.Postgres("Db", {
  region: "iad",
  migrations: "./migrations",
  importFiles: ["./seed.sql"],
});
```

MPG is on the org private network. Deploy-time apply needs a
route to that network.

## Plan

`plan` is the hardware size: `basic`, `starter`, `launch`,
`scale`, `performance`. Default is `basic`.

```typescript
const db = yield* Fly.Postgres("Db", {
  region: "iad",
  plan: "starter",
});
```

:::note[Create-only]
Changing `plan` later is ignored. Fly has no cluster update API.
:::

## Volume size

`volumeSizeGb` is the initial disk. Fly defaults to 10 GB.

```typescript
const db = yield* Fly.Postgres("Db", {
  region: "iad",
  volumeSizeGb: 20,
});
```

:::note[Create-only]
Changing `volumeSizeGb` later is ignored.
:::

## PostGIS

`postgis: true` enables PostGIS at create.

```typescript
const db = yield* Fly.Postgres("Db", {
  region: "iad",
  postgis: true,
});
```

:::note[Create-only]
Flipping `postgis` later is ignored.
:::

## Where next

[Regions](https://alchemy.run/fly/compute/regions) lists codes. Default is `iad`.
[Services](https://alchemy.run/fly/compute/services) is the always-on Machine that
binds the cluster.
[Drizzle on Postgres](../../sql/drizzle/postgres.md) is the schema-to-query
flow.
[Secrets](https://alchemy.run/fly/data/secrets) covers `Config.Redacted` for values
from `.env`.
The [`Postgres` reference](https://alchemy.run/providers/fly/reference/postgres#postgres) lists every
prop.
The [`ConnectPostgres` reference](https://alchemy.run/providers/fly/reference/postgres#connectpostgres)
is the runtime binding.
[Example: fly-postgres](https://github.com/alchemy-run/alchemy/tree/main/examples/fly-postgres)
is the complete runnable project.
