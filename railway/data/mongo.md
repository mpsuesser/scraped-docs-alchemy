---
url: https://alchemy.run/railway/data/mongo
title: "Mongo"
description: "Official MongoDB in a Project. Bind Railway.ConnectMongo and ping with the MongoDB driver."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A [`Railway.Mongo`](https://alchemy.run/providers/railway/reference/mongodb#mongo) is MongoDB as a Service:
the official `mongo` image, a Volume at `/data/db`, `MONGO_*` /
`MONGO_URL` variables, IPv6 bind for the private network, and an
optional TCP proxy for the public URL. The IaC helper is
`Railway.mongo`.

Private hostname is `{name}.railway.internal`. From a
[Service](../compute/services.md), yield
[`ConnectMongo`](https://alchemy.run/providers/railway/reference/mongodb#connectmongo). From a laptop, use
`publicConnectionUri`.

## Create Mongo

Pass a Project. Alchemy generates a unique name, password, volume,
and a public TCP proxy.

```typescript
const site = yield* Railway.Project("Site");
const db = yield* Railway.mongo("Db", { project: site });
```

:::caution[Changing `project` replaces Mongo]
A new service + volume are created in the new Project. The old
service and volume are deleted. Data is not copied.
:::

## Connect from a Service

Yield `ConnectMongo` in the Service's constructor. Provide `ConnectMongoHttp`.
Pass `conn.connectionString` to the MongoDB driver (or
`Railway.pingMongo`).

```typescript
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import * as Redacted from "effect/Redacted";

export default class Api extends Railway.Service<Api>()(
  "Api",
  {
    project: Site,
    main: import.meta.url,
    build: { install: ["mongodb"] },
  },
  Effect.gen(function* () {
    const conn = yield* Railway.ConnectMongo(Db);
    return {
      fetch: Effect.gen(function* () {
        const url = yield* conn.connectionString;
        const ping = yield* Railway.pingMongo(Redacted.value(url));
        return HttpServerResponse.json(ping);
      }),
    };
  }).pipe(Effect.provide(Railway.ConnectMongoHttp)),
) {}
```

To store Railway's `${{Db.MONGO_URL}}` template instead of a
resolved URI, pass [`Railway.ref`](https://alchemy.run/providers/railway/reference/project#ref)
(`Railway.ref(Db, "MONGO_URL")`) as a
[Variable](variables.md) `value`.

## Public TCP

`public` (default `true`) creates a TCP proxy on 27017.
`publicConnectionUri` is `{domain}:{proxyPort}` for laptop access.

```typescript
const db = yield* Railway.mongo("Db", {
  project: site,
  public: false,
});
```

## Where next

[Postgres](postgres.md) and [MySQL](mysql.md)
are the same shape for SQL.
[TcpProxy](../networking.md#tcp-proxies) is the public TCP
resource.
The [`Mongo` reference](https://alchemy.run/providers/railway/reference/mongodb#mongo) lists every prop.
The [`ConnectMongo` reference](https://alchemy.run/providers/railway/reference/mongodb#connectmongo) is
the runtime binding.
