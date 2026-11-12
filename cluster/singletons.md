---
url: https://alchemy.run/cluster/singletons
title: "Singletons"
description: "Run one fiber across the whole cluster with an Effect Cluster singleton — on exactly one runner, moved when that runner stops."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A singleton is one fiber that runs on exactly one runner. When that runner stops, another runner starts it. Use it for work that must not run twice at once: a poller, a leader’s loop, a queue consumer.

<svg viewBox="0 0 655.2 124" style="max-width: 753px" role="img" aria-label="The Sweeper runs on Runner 1 until it stops. Once its shard lock lapses, Runner 2 takes the shard and starts the fiber. It never runs on both."><line stroke-opacity="0.2" stroke="currentColor" x1="363.2" y1="24" x2="363.2" y2="118"></line><g><text fill="currentColor" x="4" y="56">Runner 1</text> <g><rect stroke="currentColor" fill="none" x="83.2" y="40" width="280" height="24" rx="5"></rect><text fill="currentColor" x="223.2" y="56" text-anchor="middle">Sweeper</text></g> <g><rect stroke="currentColor" fill="none" x="363.2" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="419.2" y="56" text-anchor="middle">stopped</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="655.2" y2="74"></line><text fill="currentColor" x="4" y="100">Runner 2</text> <g><rect stroke="currentColor" fill="none" x="447.2" y="84" width="196.00000000000006" height="24" rx="5"></rect><text fill="currentColor" x="545.2" y="100" text-anchor="middle">Sweeper</text></g></g> <text fill="currentColor" x="363.2" y="18" text-anchor="middle">Runner 1 stops</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="The Sweeper runs on Runner 1 until it stops. Once its shard lock lapses, Runner 2 takes the shard and starts the fiber. It never runs on both."><line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="24" x2="159" y2="138"></line><g><text fill="currentColor" x="0" y="44">Runner 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="159" height="24" rx="5"></rect><text fill="currentColor" x="79.5" y="70" text-anchor="middle">Sweeper</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="54" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="190.8" y="70" text-anchor="middle">stopped</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">Runner 2</text> <g><rect stroke="currentColor" fill="none" x="206.70000000000002" y="108" width="111.29999999999998" height="24" rx="5"></rect><text fill="currentColor" x="262.35" y="124" text-anchor="middle">Sweeper</text></g></g> <text fill="currentColor" x="159" y="18" text-anchor="middle">Runner 1 stops</text></svg>

The Sweeper runs on Runner 1 until it stops. Once its shard lock lapses, Runner 2 takes the shard and starts the fiber. It never runs on both.

## Declare and implement it

A singleton is declared by name, with an Effect that resolves bindings once per runner and returns the fiber:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Effect from "effect/Effect";
import * as Schedule from "effect/Schedule";
import { Default } from "./Cluster.ts";
import Room from "./Room.ts";

export default class Sweeper extends Cluster.Singleton<Sweeper>()(
  "Sweeper",
  { main: import.meta.url },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return Effect.gen(function* () {
      const lobby = yield* rooms.getByName("lobby");
      const members = yield* lobby.Members();
      yield* Effect.logInfo(\`lobby has ${members.length} members\`);
    }).pipe(Effect.repeat(Schedule.spaced("1 minute")), Effect.orDie);
  }).pipe(Effect.provide(Default)),
) {}
```

Every runner builds the outer Effect. Only the runner that owns the singleton runs the fiber it returns.

## Deploy it

Nothing calls a singleton, so nothing else yields it. `yield*` it in the Stack:

```typescript
import * as Alchemy from "alchemy";
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Api from "./src/Api.ts";
import Sweeper from "./src/Sweeper.ts";

export default Alchemy.Stack(
  "Rooms",
  { providers: Layer.mergeAll(Docker.providers(), Cluster.providers()) },
  Effect.gen(function* () {
    yield* Sweeper;
    const api = yield* Api;
    return { api: api.name };
  }),
);
```

Runners hand a singleton over through their shard locks, so while shards settle, after a deploy or when a runner starts, it can move once or twice. It never runs on two runners at once.

## Declare it on its own

A singleton can also be declared by name alone and implemented with `.make`, which returns a Layer, as an [entity](entities.md#keep-its-code-out-of-its-callers) split from its callers is. Nothing calls a singleton, so there is no caller to keep its code from; split it when you keep every implementation as a Layer:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";

export class Sweeper extends Cluster.Singleton<Sweeper>()("Sweeper") {}
```

```typescript
import * as Effect from "effect/Effect";
import * as Schedule from "effect/Schedule";
import { Default } from "./Cluster.ts";
import Room from "./Room.ts";
import { Sweeper } from "./Sweeper.ts";

export default Sweeper.make(
  { main: import.meta.url },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return Effect.gen(function* () {
      const lobby = yield* rooms.getByName("lobby");
      const members = yield* lobby.Members();
      yield* Effect.logInfo(\`lobby has ${members.length} members\`);
    }).pipe(Effect.repeat(Schedule.spaced("1 minute")), Effect.orDie);
  }).pipe(Effect.provide(Default)),
);
```

Provide the Layer to the Stack, next to the rest of your implementations:

```typescript
import * as Alchemy from "alchemy";
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Api from "./src/Api.ts";
import SweeperLive from "./src/SweeperLive.ts";

export default Alchemy.Stack(
  "Rooms",
  { providers: Layer.mergeAll(Docker.providers(), Cluster.providers()) },
  Effect.gen(function* () {
    const api = yield* Api;
    return { api: api.name };
  }).pipe(Effect.provide(SweeperLive)),
);
```
