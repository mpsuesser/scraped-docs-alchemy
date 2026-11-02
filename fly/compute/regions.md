---
url: https://alchemy.run/fly/compute/regions
title: "Regions"
description: "Where Fly Machines, Volumes, Postgres, and Redis live. Alchemy defaults to iad."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A region is a Fly.io datacenter. [Machines](machines.md),
[Volumes](../data/volumes.md), [Postgres](../data/postgres.md)
clusters, and a [Redis](../data/redis.md) primary live in one. An
[App](apps.md) is global. It is not pinned to a region.

Alchemy defaults to `iad` (Ashburn, Virginia). Pass `region` to put
a [Service](services.md), Machine, or Postgres cluster
somewhere else.

## Pin a Service

```typescript
export default class Api extends Fly.Service<Api>()(
  "Api",
  { main: import.meta.url, region: "lhr", port: 3000 },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello")),
    };
  }),
) {}
```

`count` Machines run in that region.

:::caution[Changing `region` moves the Machines]
The Service keeps its App and hostname. Machines are created in the
new region and the old ones are deleted. A Volume does not move with
them.
:::

## Pin a Machine

```typescript
const web = yield* Fly.Machine("Web", {
  app: Site,
  region: "syd",
  image: "nginx:alpine",
});
```

Each Machine resource is one VM in one region. To run in two
regions, declare two Machines.

:::caution[Changing `region` replaces the Machine]
The Machine is created in the new region. The old one is deleted.
:::

## Volumes stay in the region

A Volume is created in the parent Service or Machine's region. It
cannot move and cannot attach to a Machine in another region.

Restore into a new region from a
[`VolumeSnapshot`](https://alchemy.run/providers/fly/reference/volume#volumesnapshot) with `snapshotId`
on the mount.

## Several regions, one hostname

Pass a list to `region` to run the same Service in several regions
behind one hostname. `count` Machines run in each region, and Fly's
Anycast proxy sends each request to the nearest healthy Machine.

```typescript
export default class Api extends Fly.Service<Api>()(
  "Api",
  { main: import.meta.url, region: ["iad", "lhr"], count: 2 },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello")),
    };
  }),
) {}
```

This runs four Machines: two in `iad` and two in `lhr`. `api.regions`
is `["iad", "lhr"]`, and `api.url` serves from both.

Adding or removing a region updates the Service in place. Its App and
hostname stay the same, Machines in regions you keep are left alone,
and a removed region's Machines are deleted. A Volume from
`MountVolume` is created in each Machine's own region.

## Which region is this?

Fly sets `FLY_REGION` on the Machine. Read it in a Service with
`Config`.

```typescript
import * as Config from "effect/Config";

const region = yield* Config.String("FLY_REGION");
```

## Region codes

Codes are three letters. This list is what Fly published as of
August 2026. Run `fly platform regions` or see
[Fly's regions](https://fly.io/docs/reference/regions/) for the
live set.

| Code | Location |
| --- | --- |
| `ams` | Amsterdam, Netherlands |
| `arn` | Stockholm, Sweden |
| `bom` | Mumbai, India |
| `cdg` | Paris, France |
| `dfw` | Dallas, Texas (US) |
| `ewr` | Secaucus, NJ (US) |
| `fra` | Frankfurt, Germany |
| `gru` | São Paulo, Brazil |
| `iad` | Ashburn, Virginia (US) |
| `jnb` | Johannesburg, South Africa |
| `lax` | Los Angeles, California (US) |
| `lhr` | London, United Kingdom |
| `nrt` | Tokyo, Japan |
| `ord` | Chicago, Illinois (US) |
| `sin` | Singapore |
| `sjc` | San Jose, California (US) |
| `syd` | Sydney, Australia |
| `yyz` | Toronto, Canada |

A dedicated [`IpAssignment`](../networking.md) can take `region`
too. Shared Anycast (`shared_v4`) ignores it.

## Postgres is regional

The cluster lives in one region. Pass `region` on
[`Fly.Postgres`](../data/postgres.md), not on the App.

```typescript
const db = yield* Fly.Postgres("Db", {
  region: "iad",
});
```

:::caution[Changing `region` replaces the cluster]
A new cluster is created in the new region. The old cluster is
deleted. Data is not copied.
:::

## Redis primary

[`Fly.Redis`](../data/redis.md) takes `primaryRegion`. Default is
`iad`. Read replicas are extra regions on `readRegions`.

```typescript
const cache = yield* Fly.Redis("Cache", {
  primaryRegion: "iad",
  readRegions: ["sjc"],
});
```

:::caution[Changing `primaryRegion` replaces Redis]
The primary region is immutable. The old database is deleted first.
:::

## Where next

[Services](services.md) pin `region` on the class.
[Machines](machines.md) pin it on the resource.
[Volumes](../data/volumes.md) follow the parent.
[Postgres](../data/postgres.md) is a billed cluster in one region.
[Redis](../data/redis.md) pins `primaryRegion`.
