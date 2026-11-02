---
url: https://alchemy.run/railway/data/redis
title: "Redis"
description: "Redis in a Project. Bind Railway.ReadRedis / WriteRedis / ReadWriteRedis on a Service."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

[`Railway.Redis`](https://alchemy.run/providers/railway/reference/redis#redis) is a Redis container in
a [Project](../compute/projects.md) (`redis:7` or
`bitnami/redis`). Alchemy writes `REDIS_URL` / `REDISPASSWORD` on
the service. Bind [`ReadWriteRedis`](https://alchemy.run/providers/railway/reference/redis#readwriteredis)
on a [Service](../compute/services.md). Reach it from a laptop
through [`TcpProxy`](../networking.md#tcp-proxies) on port `6379`.

## Create Redis

Pass a Project. Alchemy generates a unique name and password unless
you pass them. Default image is `redis:7`.

```typescript
const site = yield* Railway.Project("Site");
const cache = yield* Railway.Redis("Cache", { project: site });
```

:::note
Prefer omitting `name` in tests and CI so names stay unique and
reclaimable.
:::

## A stable name

Pass `name` when you want a stable service name. Changing it later
updates the service in place.

```typescript
const cache = yield* Railway.Redis("Cache", {
  project: site,
  name: "cache",
});
```

## Image

Official Redis (`redis:7`) is started with `--requirepass`. Bitnami
reads `REDIS_PASSWORD`. Updating the image is in place.

```typescript
const cache = yield* Railway.Redis("Cache", {
  project: site,
  image: "bitnami/redis",
});
```

## Bind from a Service

Yield `Railway.ReadWriteRedis` (or `ReadRedis` / `WriteRedis`) in
the Service's constructor. Provide the matching `*Http` layer. Runtime commands
use `REDIS_URL` (`{name}.railway.internal`).

```typescript
import * as HttpServerResponse from "effect/http/HttpServerResponse";

const Cache = Railway.Redis("Cache", { project: Site });

export default class Api extends Railway.Service<Api>()(
  "Api",
  { project: Site, main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const cache = yield* Railway.ReadWriteRedis(Cache);
    return {
      fetch: Effect.gen(function* () {
        yield* cache.set("marker", "hello");
        const value = yield* cache.get("marker");
        return HttpServerResponse.json({ value });
      }),
    };
  }).pipe(Effect.provide(Railway.ReadWriteRedisHttp)),
) {}
```

## Public TCP

Attach a [`TcpProxy`](../networking.md#tcp-proxies) on `6379` to
reach Redis from outside the private network.

```typescript
const cache = yield* Railway.Redis("Cache", { project: site });
const proxy = yield* Railway.TcpProxy("CacheProxy", {
  redis: cache,
  environment: site,
  applicationPort: 6379,
});
```

## Password

Wrap `password` with `Redacted.make(...)`. If omitted, a password is
generated on first create and kept. Never persisted in attributes.

```typescript
import * as Redacted from "effect/Redacted";

const cache = yield* Railway.Redis("Cache", {
  project: site,
  password: Redacted.make("…"),
});
```

## Where next

[Regions](../compute/regions.md) lists codes.
[Services](../compute/services.md) is the container that binds
`ReadWriteRedis`.
The [`Redis` reference](https://alchemy.run/providers/railway/reference/redis#redis) lists every prop.
