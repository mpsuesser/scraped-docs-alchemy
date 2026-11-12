---
url: https://alchemy.run/cluster/what-is-effect-cluster
title: "What is Effect Cluster?"
description: "The concepts behind Effect Cluster — entities addressed by id, messages handled one at a time, shards, runners and pools, the database, workflows, singletons and crons — and how @alchemy.run/effect-cluster deploys them."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

[Effect Cluster](https://effect.website/) is Effect’s library for stateful services spread across many processes. Its center is the **entity**: a small, long-lived program with an id and its own state, like one chat room. Effect Cluster runs each instance on one process at a time, delivers its messages in order, and moves it when processes come and go. The processes are **runners**, and they agree on who runs what through a Postgres **database**.

Take one small app: chat rooms. Each room is an instance of the `Room` entity, and an Api container calls a room by its name. Two runners, in a **pool** called `Default`, hold the rooms between them, and the database records which runner owns each room’s shard. With Alchemy all of it deploys from one Stack, here on a Docker swarm:

Docker swarmyour cloud

Apiyour app · Docker Service

clusterrunners + database

Defaultpool

gatewayHTTP · the pool's name

Runner 1shards 1–150

lobbyRoom

Runner 2shards 151–300

kitchenRoom

DatabasePostgres

messages

persisted

Api is your own service. It calls a room through the pool's gateway, reached by the pool's name, which routes the call to the runner holding the room. Each room runs inside the runner that owns its shard. The database records the owners, so either runner can take over the other's shards.

## Entities: a class and its instances

An entity starts from the messages it accepts, as Effect RPC schemas, passed to Effect’s `Entity.make`:

```typescript
import { Entity } from "effect/cluster";
import { Rpc } from "effect/rpc";
import * as Schema from "effect/Schema";

export const Rooms = Entity.make("Room", [
  Rpc.make("Join", { payload: { name: Schema.String }, success: Schema.Int }),
  Rpc.make("Members", { success: Schema.Array(Schema.String) }),
]);
```

An **instance** is one room: the lobby, the kitchen. Every instance runs the same code against its own state.

See [Entities](entities.md).

## One instance per id, one message at a time

You never create an instance. Every id exists from the start, and the first message to an id brings its instance to life. Inside any runtime’s Effect, `yield* Room` (the class [below](#an-instances-code-and-its-state)) returns the entity’s client. A caller gets a room by its name and sends it messages:

```typescript
const rooms = yield* Room;
const lobby = yield* rooms.getByName("lobby");
yield* lobby.Join({ name: "sam" });
```

However many callers message the lobby, from however many servers, they reach the same instance, and it handles their messages one at a time. Other rooms run beside it:

<svg viewBox="0 0 647.8 102" style="max-width: 745px" role="img" aria-label="Sam and Ada join the lobby at the same moment from two servers. The lobby takes Join sam, then Join ada. The kitchen handles its own message meanwhile."><g><text fill="currentColor" x="4" y="34">lobby</text> <g><rect stroke="currentColor" fill="none" x="98.20000000000002" y="18" width="145.6" height="24" rx="5"></rect><text fill="currentColor" x="171" y="34" text-anchor="middle">Join sam</text></g> <g><rect stroke="currentColor" fill="none" x="243.8" y="18" width="145.59999999999997" height="24" rx="5"></rect><text fill="currentColor" x="316.6" y="34" text-anchor="middle">Join ada</text></g> <g><rect stroke="currentColor" fill="none" x="389.4" y="18" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="456.59999999999997" y="34" text-anchor="middle">Members</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="647.8" y2="52"></line><text fill="currentColor" x="4" y="78">kitchen</text> <g><rect stroke="currentColor" fill="none" x="143" y="62" width="145.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="215.8" y="78" text-anchor="middle">Join kim</text></g></g></svg> <svg viewBox="0 0 330 122" style="max-width: 379px" role="img" aria-label="Sam and Ada join the lobby at the same moment from two servers. The lobby takes Join sam, then Join ada. The kitchen handles its own message meanwhile."><g><text fill="currentColor" x="0" y="22">lobby</text> <g><rect stroke="currentColor" fill="none" x="12.72" y="32" width="82.67999999999999" height="24" rx="5"></rect><text fill="currentColor" x="54.059999999999995" y="48" text-anchor="middle">Join sam</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="32" width="82.67999999999999" height="24" rx="5"></rect><text fill="currentColor" x="136.73999999999998" y="48" text-anchor="middle">Join ada</text></g> <g><rect stroke="currentColor" fill="none" x="178.07999999999998" y="32" width="76.32000000000002" height="24" rx="5"></rect><text fill="currentColor" x="216.24" y="48" text-anchor="middle">Members</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">kitchen</text> <g><rect stroke="currentColor" fill="none" x="38.16" y="86" width="82.68" height="24" rx="5"></rect><text fill="currentColor" x="79.5" y="102" text-anchor="middle">Join kim</text></g></g></svg>

Sam and Ada join the lobby at the same moment from two servers. The lobby takes Join sam, then Join ada. The kitchen handles its own message meanwhile.

That’s what lets a room keep its members in a plain `Ref` without locks: nothing else ever touches them.

## An instance’s code and its state

`Cluster.Entity` makes the entity a class, from its messages and its code. The code comes in two phases. The outer Effect runs once per runner and resolves what every room shares, here a capacity from config. The inner one runs once per instance, on its first message: it creates that room’s own members and returns the handlers for its messages:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";
import * as Ref from "effect/Ref";
import { Default } from "./Cluster.ts";
import { Rooms } from "./Rooms.ts";

export default class Room extends Cluster.Entity<Room>()(
  Rooms,
  { main: import.meta.url },
  Effect.gen(function* () {
    const capacity = yield* Config.Int("ROOM_CAPACITY");
    return Effect.gen(function* () {
      const members = yield* Ref.make(new Set<string>());
      return Rooms.of({
        Join: ({ payload }) =>
          Ref.modify(members, (all) => {
            if (all.size >= capacity) return [all.size, all];
            const next = new Set(all).add(payload.name);
            return [next.size, next];
          }),
        Members: () => Effect.map(Ref.get(members), (all) => [...all]),
      });
    });
  }).pipe(Effect.provide(Default)),
) {}
```

`Rooms.of` types the handlers against the messages. `Default` is the pool the room runs on, [next](#runners-run-the-instances).

An instance’s state lives in memory. When it sits idle (a minute, by default), when its runner restarts, or when a deploy replaces the runners, the instance stops and its state is gone. The next message starts it fresh. Keep anything that must survive in a database the instance reads on its first message.

## Runners run the instances

**Runners** are the processes that host instances. A **pool** is a group of runners that run the same code, and `Default` is one:

```typescript
import * as ClusterDocker from "@alchemy.run/effect-cluster/Docker";
import * as Layer from "effect/Layer";

export const Database = ClusterDocker.Database("Cluster");

export const Default = ClusterDocker.SwarmPool("Default", { replicas: 2 }).pipe(
  Layer.provide(Database),
);
```

Providing `Default` to the room’s code is what deploys it. Every class given the same pool is bundled into one program, built into one image, and run on every runner in the pool. A second pool is its own **shard group**: its classes only run on its runners.

See [Pools](pools.md).

## Shards and the database

Effect Cluster hashes each instance’s id to one of 300 **shards**, and each runner owns some of the shards. A message to the lobby goes to whichever runner owns the lobby’s shard.

The runners agree on who owns what through the **database**: a Postgres they all reach. Each runner registers itself there and takes locks on its shards. When a runner stops, its locks lapse, and the others take its shards over, along with the instances in them.

<svg viewBox="0 0 726 112" style="max-width: 835px" role="img" aria-label="An id always hashes to the same shard; the shard's owner can change."><defs><marker id="alc-dag-arrow-tb5qpg" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 56 C 225.5 56, 225.5 56, 287 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-tb5qpg)"></path></g><g><path d="M 433 56 C 494.5 56, 494.5 56, 556 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-tb5qpg)"></path><text fill="currentColor" x="494.5" y="50" text-anchor="middle">lock in Postgres</text></g> <g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">lobby</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">id</text></g> <g transform="translate(293, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">shard 42</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">hash of the id</text></g> <g transform="translate(562, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Runner 1</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">owns shard 42</text></g></svg> <svg viewBox="0 0 129 332" style="max-width: 148px" role="img" aria-label="An id always hashes to the same shard; the shard's owner can change."><defs><marker id="alc-dag-arrow-wqexc5" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 64.5 74 C 64.5 101, 64.5 101, 64.5 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-wqexc5)"></path></g><g><path d="M 64.5 198 C 64.5 225, 64.5 225, 64.5 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-wqexc5)"></path><text fill="currentColor" x="64.5" y="219" text-anchor="middle">lock in Postgres</text></g> <g transform="translate(10, 10)"><rect stroke="currentColor" fill="none" width="109" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="54.5" y="28" text-anchor="middle" dominant-baseline="middle">lobby</text> <text fill="currentColor" x="54.5" y="46" text-anchor="middle" dominant-baseline="middle">id</text></g> <g transform="translate(10, 134)"><rect stroke="currentColor" fill="none" width="109" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="54.5" y="28" text-anchor="middle" dominant-baseline="middle">shard 42</text> <text fill="currentColor" x="54.5" y="46" text-anchor="middle" dominant-baseline="middle">hash of the id</text></g> <g transform="translate(10, 258)"><rect stroke="currentColor" fill="none" width="109" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="54.5" y="28" text-anchor="middle" dominant-baseline="middle">Runner 1</text> <text fill="currentColor" x="54.5" y="46" text-anchor="middle" dominant-baseline="middle">owns shard 42</text></g></svg>

An id always hashes to the same shard; the shard's owner can change.

See [Databases](databases.md).

## Persisted messages

By default a message travels straight to its runner. Mark an RPC **persisted** with the `Persisted` annotation from `ClusterSchema`, Effect Cluster’s RPC annotations, and the message is written to the database first, delivered at least once, and handled once per primary key:

```typescript
import { ClusterSchema, Entity } from "effect/cluster";
import { Rpc } from "effect/rpc";
import * as Schema from "effect/Schema";

export const Rooms = Entity.make("Room", [
  Rpc.make("Post", {
    payload: { messageId: Schema.String, text: Schema.String },
    primaryKey: ({ messageId }) => messageId,
  }).annotate(ClusterSchema.Persisted, true),
]);
```

A persisted message survives its runner restarting: whichever runner owns the shard next delivers it.

## The gateway

Callers outside the cluster, like a Worker or a Lambda function, don’t reach the database or the runners. They reach the **gateway**: each pool serves its classes over HTTP, behind a token every caller is given. On AWS, GCP and Hetzner a load balancer puts it on the internet; on Fly it’s the pool’s `fly.dev` address. On [Cloudflare](cloudflare.md) the pool is one Worker and each instance is a Durable Object, so there are no shards and no database.

Inside the cluster, one entity calling another skips the gateway: the call goes straight from runner to runner.

See [Calling an entity](calling.md).

## Workflows, singletons and crons

Effect Cluster runs three more kinds of programs in the same pool, declared the same way as an entity. A **workflow** is a durable function: each step is persisted, it can sleep for days or wait for a signal, and it picks up where it left off on another runner. A **singleton** is one fiber running on exactly one runner.

A **cron** is a job that runs on a schedule, once per scheduled time across the cluster, however many runners the pool has:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Effect from "effect/Effect";
import { Default } from "./Cluster.ts";

export default class Cleanup extends Cluster.Cron<Cleanup>()(
  "Cleanup",
  { main: import.meta.url, cron: "0 3 * * *" },
  Effect.gen(function* () {
    return Effect.logInfo("closing empty rooms");
  }).pipe(Effect.provide(Default)),
) {}
```

See [Workflows](workflows.md), [Singletons](singletons.md) and [Crons](crons.md).

## How @alchemy.run/effect-cluster models these concepts

Each cloud’s entry point, like `@alchemy.run/effect-cluster/Docker`, has a `Database` Layer that runs Postgres on that cloud and a pool Layer that runs the runners: `SwarmPool`, `EcsPool`, `ComputeEnginePool`, `DeploymentPool`, `MachinePool` or `ServerPool`. `Cluster.Entity`, `Cluster.Workflow`, `Cluster.Singleton` and `Cluster.Cron` are classes, and providing a pool to one deploys it. `yield* Room` from any runtime returns a client that reaches the pool’s gateway, with its URL and token given to that runtime. On Cloudflare, `DurableObjectPool` runs each instance as a Durable Object instead.

Next, [Setup](setup.md), then the [tutorial](tutorial/part-1.md).
