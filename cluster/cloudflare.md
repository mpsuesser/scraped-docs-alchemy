---
url: https://alchemy.run/cluster/cloudflare
title: "Cloudflare"
description: "Run an Effect Cluster's entities, workflows, singletons and crons on Cloudflare Durable Objects, with no database and no servers to manage."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Host your cluster on Cloudflare: one Worker runs every class you give it, on Durable Objects. There is no database and nothing to scale. Callers reach it through the Worker’s gateway.

```typescript
import * as ClusterCloudflare from "@alchemy.run/effect-cluster/Cloudflare";

export const Default = ClusterCloudflare.DurableObjectPool("Default");
```

Install `@alchemy.run/cloudflare` and finish [Cloudflare setup](../cloudflare/setup.md). Add its providers next to the cluster’s:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Api from "./src/Api.ts";
import RoomLive from "./src/RoomLive.ts";

export default Alchemy.Stack(
  "Rooms",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), Cluster.providers()),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.url };
  }).pipe(Effect.provide(RoomLive)),
);
```

Your entities, workflows, singletons and crons don’t change. Their implementations take `Default` like they take any pool.

## Each instance is a Durable Object

A Durable Object is already one instance per name, in one place, handling one event at a time. That’s what Effect Cluster’s shards and runners exist to give you, so on Cloudflare there are none. Every entity type shares one Durable Object class, and each instance is the object named by its type and id. This is the design of Effect’s own Durable Object support.

<svg viewBox="0 0 814 204" style="max-width: 936px" role="img" aria-label="The Worker routes each call to the object named by the entity's type and id."><defs><marker id="alc-dag-arrow-h2v4mr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 210 102 C 259 102, 259 102, 308 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-h2v4mr)"></path><text fill="currentColor" x="259" y="96" text-anchor="middle">lobby.Join()</text></g> <g><path d="M 500 102 C 549 102, 549 56, 598 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-h2v4mr)"></path></g><g><path d="M 500 102 C 549 102, 549 148, 598 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-h2v4mr)"></path></g><g transform="translate(24, 70)"><rect stroke="currentColor" fill="none" width="186" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="93" y="28" text-anchor="middle" dominant-baseline="middle">Callers</text> <text fill="currentColor" x="93" y="46" text-anchor="middle" dominant-baseline="middle">Worker, Lambda, container</text></g> <g transform="translate(314, 70)"><rect stroke="currentColor" fill="none" width="186" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="93" y="28" text-anchor="middle" dominant-baseline="middle">Pool Worker</text> <text fill="currentColor" x="93" y="46" text-anchor="middle" dominant-baseline="middle">the gateway</text></g> <g transform="translate(604, 24)"><rect stroke="currentColor" fill="none" width="186" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="93" y="28" text-anchor="middle" dominant-baseline="middle">Room lobby</text> <text fill="currentColor" x="93" y="46" text-anchor="middle" dominant-baseline="middle">Durable Object</text></g> <g transform="translate(604, 116)"><rect stroke="currentColor" fill="none" width="186" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="93" y="28" text-anchor="middle" dominant-baseline="middle">Room kitchen</text> <text fill="currentColor" x="93" y="46" text-anchor="middle" dominant-baseline="middle">Durable Object</text></g></svg> <svg viewBox="0 0 404 332" style="max-width: 465px" role="img" aria-label="The Worker routes each call to the object named by the entity's type and id."><defs><marker id="alc-dag-arrow-gx7hw1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 202 74 C 202 101, 202 101, 202 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-gx7hw1)"></path><text fill="currentColor" x="202" y="95" text-anchor="middle">lobby.Join()</text></g> <g><path d="M 202 198 C 202 225, 99 225, 99 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-gx7hw1)"></path></g><g><path d="M 202 198 C 202 225, 305 225, 305 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-gx7hw1)"></path></g><g transform="translate(113, 10)"><rect stroke="currentColor" fill="none" width="178" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="89" y="28" text-anchor="middle" dominant-baseline="middle">Callers</text> <text fill="currentColor" x="89" y="46" text-anchor="middle" dominant-baseline="middle">Worker, Lambda, container</text></g> <g transform="translate(113, 134)"><rect stroke="currentColor" fill="none" width="178" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="89" y="28" text-anchor="middle" dominant-baseline="middle">Pool Worker</text> <text fill="currentColor" x="89" y="46" text-anchor="middle" dominant-baseline="middle">the gateway</text></g> <g transform="translate(10, 258)"><rect stroke="currentColor" fill="none" width="178" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="89" y="28" text-anchor="middle" dominant-baseline="middle">Room lobby</text> <text fill="currentColor" x="89" y="46" text-anchor="middle" dominant-baseline="middle">Durable Object</text></g> <g transform="translate(216, 258)"><rect stroke="currentColor" fill="none" width="178" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="89" y="28" text-anchor="middle" dominant-baseline="middle">Room kitchen</text> <text fill="currentColor" x="89" y="46" text-anchor="middle" dominant-baseline="middle">Durable Object</text></g></svg>

The Worker routes each call to the object named by the entity's type and id.

One entity calling another goes straight from object to object, without the gateway.

## State lasts until the instance is idle

An instance keeps its state in memory, as on runners, until it has been idle for its [`maxIdleTime`](entities.md#how-long-an-instance-lives). Cloudflare puts an object to sleep after about ten seconds without events, so until then the object’s alarm wakes it every few seconds. A short `maxIdleTime` costs fewer alarms:

```typescript
import * as Effect from "effect/Effect";
import * as Ref from "effect/Ref";
import { Default } from "./Cluster.ts";
import { Room } from "./Room.ts";

export default Room.make(
  { main: import.meta.url, maxIdleTime: "15 seconds" },
  Effect.gen(function* () {
    return Effect.gen(function* () {
      const members = yield* Ref.make(new Set<string>());
      return Room.of({
        Join: ({ payload }) =>
          Ref.modify(members, (all) => {
            const next = new Set(all).add(payload.name);
            return [next.size, next];
          }),
        Members: () => Effect.map(Ref.get(members), (all) => [...all]),
      });
    });
  }).pipe(Effect.provide(Default)),
);
```

An instance held with `Entity.keepAlive(true)` doesn’t stop when idle. If its object restarts, the alarm starts the instance again.

## A deploy reaches an instance when its object restarts

A deploy gives the pool’s Worker a new version. An object that is asleep starts on the new version at its next call. An object that is awake keeps its instance, its in-memory state and the version it started on until Cloudflare restarts it, which can take several minutes.

So after a deploy, an instance you hold with `Entity.keepAlive(true)` may answer with the old code for a while. Persisted messages, delayed messages and cron times are in the object’s SQLite, so they are delivered whichever version the object is on.

## Handlers run as on runners

Each instance runs its handlers behind Effect’s own RPC server, the way Effect Cluster’s runners do. RPC middleware, `concurrency`, `Rpc.fork` and `Rpc.uninterruptible` all work as they do on runners. When a caller is interrupted, its handler is interrupted too.

A defect restarts the instance and runs its messages again, with a growing pause between restarts. To send the defect to the caller instead, set `disableFatalDefects: true`.

## Persisted messages live in the object

A persisted message is written to the object’s own SQLite before it runs, and its reply after. A message with a primary key runs once. A duplicate that arrives while the first is running waits for its reply. A message cut short by a restart runs again when the object’s alarm fires. A message with `DeliverAt` waits in SQLite until its time, and the alarm wakes the object to run it.

A call the object never answers fails with `PersistenceError`. A persisted call is first sent again to a fresh stub, since the object stores it under the same id and runs it once.

## Workflows, singletons and crons

Each workflow execution is a Durable Object of its own, running Alchemy’s Durable Object [workflow engine](../cloudflare/compute/effect-workflows.md): activities, durable sleeps and signals are kept in its SQLite.

A singleton runs in an object that its alarm checks on every few seconds, restarting the fiber if the object restarted. A cron runs in an object that its alarm wakes at each scheduled time, down to the second.

## Call it from a Worker

A Worker can’t fetch another Worker’s `workers.dev` address in the same account, so a Worker that yields one of your classes gets a service binding to the pool’s Worker, and its calls go through that:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Room } from "./Room.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return {
      fetch: Effect.gen(function* () {
        const lobby = yield* rooms.getByName("lobby");
        return yield* HttpServerResponse.json({ members: yield* lobby.Join({ name: "sam" }) });
      }).pipe(Effect.orDie),
    };
  }),
) {}
```

Callers on any other runtime, like a Lambda function or a container, use the Worker’s `workers.dev` address, with the token Alchemy binds into them.

## What doesn’t run on Cloudflare

An [existing Effect Cluster program](existing-programs.md) needs Node: give it a pool on another host. Shard groups don’t apply, because there are no shards.

Every option, with its default, is in the reference: [DurableObjectPool](https://alchemy.run/providers/cluster/cloudflare#durableobjectpool).
