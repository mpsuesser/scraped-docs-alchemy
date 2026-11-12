---
url: https://alchemy.run/cluster
title: "Effect Cluster"
description: "Run Effect Cluster entities, workflows, singletons and crons with Alchemy — a Postgres database and a pool of runners on Docker, AWS, GCP, Kubernetes, Fly or Hetzner, or Durable Objects on Cloudflare."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

[Effect Cluster](https://effect.website/) runs stateful, addressable programs across a group of processes. An **entity** has one instance per id, like a chat room or a cart, and handles its messages one at a time, so it keeps state without locks. Instances are spread over **runners** by shard, and a Postgres database records which runner owns each shard and keeps the messages that must survive a restart. **Workflows** persist each step and resume on another runner, **singletons** run one fiber across the cluster, and **crons** run once per scheduled time.

With Alchemy, the database and a **pool** of runners deploy in the same Stack as the rest of your app, on Docker, AWS, GCP, Kubernetes, Fly or Hetzner. On Cloudflare there is no database: each instance is a Durable Object.

**New to Effect Cluster?** Start with [What is Effect Cluster?](cluster/what-is-effect-cluster.md).

## A quick example

A pool of runners, and the database they coordinate through:

```typescript
import * as ClusterDocker from "@alchemy.run/effect-cluster/Docker";
import * as Layer from "effect/Layer";

export const Database = ClusterDocker.Database("Cluster");

export const Default = ClusterDocker.SwarmPool("Default").pipe(Layer.provide(Database));
```

An entity, declared with its messages and its implementation, on that pool:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import { Entity } from "effect/cluster";
import * as Effect from "effect/Effect";
import * as Ref from "effect/Ref";
import { Rpc } from "effect/rpc";
import * as Schema from "effect/Schema";
import { Default } from "./Cluster.ts";

const Rooms = Entity.make("Room", [
  Rpc.make("Join", { payload: { name: Schema.String }, success: Schema.Int }),
]);

export default class Room extends Cluster.Entity<Room>()(
  Rooms,
  { main: import.meta.url },
  Effect.gen(function* () {
    return Effect.gen(function* () {
      const members = yield* Ref.make(new Set<string>());
      return Rooms.of({
        Join: ({ payload }) =>
          Ref.modify(members, (all) => {
            const next = new Set(all).add(payload.name);
            return [next.size, next];
          }),
      });
    });
  }).pipe(Effect.provide(Default)),
) {}
```

Any runtime calls a room by its id:

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import Room from "./Room.ts";

export default class Api extends Docker.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return {
      fetch: Effect.gen(function* () {
        const lobby = yield* rooms.getByName("lobby");
        const members = yield* lobby.Join({ name: "sam" });
        return yield* HttpServerResponse.json({ members });
      }).pipe(Effect.orDie),
    };
  }),
) {}
```

Swap `ClusterDocker` for `ClusterAWS`, `ClusterFly` or another cloud, and the room doesn’t change.

## Where to start

**Want to build?** Start with [Setup](cluster/setup.md), then the [tutorial](cluster/tutorial/part-1.md) builds chat rooms step by step, from a room on your machine to invitations, a nightly cleanup and AWS.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Running

- [Databases](cluster/databases.md)
- [Pools](cluster/pools.md)

### Platforms

- [Docker](cluster/docker.md)
- [AWS](cluster/aws.md)
- [GCP](cluster/gcp.md)
- [Kubernetes](cluster/kubernetes.md)
- [Fly](cluster/fly.md)
- [Hetzner](cluster/hetzner.md)
- [Cloudflare](cluster/cloudflare.md)
