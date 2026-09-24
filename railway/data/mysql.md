---
url: https://alchemy.run/railway/data/mysql
title: "MySQL"
description: "Official MySQL in a Project. Bind Railway.ConnectMySQL and query with Drizzle or SQL."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

A [`Railway.MySQL`](https://alchemy.run/providers/railway/reference/mysql#mysql) is MySQL as a Service:
the official `mysql:9` image, a Volume at `/var/lib/mysql`, `MYSQL_*`
/ `MYSQL_URL` variables, and an optional TCP proxy for the public URL.
The IaC helper is `Railway.mysql`.

Private hostname is `{name}.railway.internal`. From a
[Service](https://alchemy.run/railway/compute/services), yield
[`ConnectMySQL`](https://alchemy.run/providers/railway/reference/mysql#connectmysql). From a laptop, use
`publicConnectionUri`.

## Create MySQL

Pass a Project. Alchemy generates a unique name, password, volume,
and a public TCP proxy.

```typescript
const site = yield* Railway.Project("Site");
const db = yield* Railway.MySQL("Db", { project: site });
```

:::caution[Changing `project` replaces MySQL]
A new service + volume are created in the new Project. The old
service and volume are deleted. Data is not copied.
:::

## Connect from a Service

Yield `ConnectMySQL` in the Service's constructor. Provide
`ConnectMySQLHttp`. Pass `conn.connectionString` to Drizzle or SQL.

```typescript
import * as Drizzle from "alchemy/Drizzle/MySQL";
import * as HttpServerResponse from "effect/unstable/http/HttpServerResponse";

export default class Api extends Railway.Service<Api>()(
  "Api",
  {
    project: Site,
    main: import.meta.url,
    build: { install: ["mysql2"] },
  },
  Effect.gen(function* () {
    const conn = yield* Railway.ConnectMySQL(Db);
    const db = yield* Drizzle.MySQL(conn.connectionString);
    return {
      fetch: Effect.gen(function* () {
        const rows = yield* db.execute("select 1 as ok", "objects");
        return HttpServerResponse.json({ rows });
      }),
    };
  }).pipe(Effect.provide(Railway.ConnectMySQLHttp)),
) {}
```

`connectionString` is the private URI
(`{name}.railway.internal:3306`). Packed into the Service env — not
`Config.Redacted`.

To store Railway's `${{Db.MYSQL_URL}}` template instead of a
resolved URI, pass [`Railway.ref`](https://alchemy.run/providers/railway/reference/project#ref)
(`Railway.ref(Db, "MYSQL_URL")`) as a
[Variable](https://alchemy.run/railway/data/variables) `value`.

## Public TCP

`public` (default `true`) creates a TCP proxy on 3306.
`publicConnectionUri` is `{domain}:{proxyPort}` for laptop access.

```typescript
const db = yield* Railway.MySQL("Db", {
  project: site,
  public: false,
});
```

## Where next

[Postgres](postgres.md) is the same shape for Postgres.
[Mongo](https://alchemy.run/railway/data/mongo) is MongoDB.
[TcpProxy](https://alchemy.run/railway/networking#tcp-proxies) is the public TCP
resource.
The [`MySQL` reference](https://alchemy.run/providers/railway/reference/mysql#mysql) lists every prop.
The [`ConnectMySQL` reference](https://alchemy.run/providers/railway/reference/mysql#connectmysql) is
the runtime binding.
