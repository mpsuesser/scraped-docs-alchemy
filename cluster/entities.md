---
url: https://alchemy.run/cluster/entities
title: "Entities"
description: "How an Effect Cluster entity is written in Alchemy — its declaration, its two-phase implementation, its instances, and where its code lives."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

An entity is a class with one instance per id. Every instance runs the same code against its own state, and handles its messages one at a time.

Each instance has a mailbox. Messages to one id queue in the order they arrive, and the instance takes the next only when it has finished the last. Two instances never wait on each other:

<svg viewBox="0 0 743.8 168" style="max-width: 855px" role="img" aria-label="The lobby handles Join sam while Join ada and Members wait behind it. The kitchen handles its own message at the same time."><defs><marker id="alc-q-arrow-rl56lf" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="8" y="71" width="100" height="46" rx="8"></rect><text fill="currentColor" x="58" y="98" text-anchor="middle">callers</text> <line stroke-opacity="0.2" stroke="currentColor" x1="108" y1="94" x2="189.4" y2="94" marker-end="url(#alc-q-arrow-rl56lf)"></line><text fill="currentColor" x="149.7" y="86" text-anchor="middle">getByName</text></g> <rect stroke="currentColor" fill="none" x="191.4" y="30" width="340.59999999999997" height="128" rx="10"></rect><text fill="currentColor" x="195.4" y="20">Room · two instances</text> <text fill="currentColor" x="207.4" y="61">lobby</text> <text fill="currentColor" x="207.4" y="117">kitchen</text> <g><path stroke="currentColor" fill="none" d="M 516 57 C 582 57, 572 53, 620 53" marker-end="url(#alc-q-arrow-rl56lf)"></path><rect stroke="currentColor" fill="none" x="443.2" y="42" width="72.8" height="30" rx="5"></rect><text fill="currentColor" x="479.59999999999997" y="61" text-anchor="middle" opacity="0.75">Join sam</text></g> <g><rect stroke="currentColor" fill="none" x="356.4" y="42" width="72.8" height="30" rx="5"></rect><text fill="currentColor" x="392.79999999999995" y="61" text-anchor="middle" opacity="1">Join ada</text></g> <g><rect stroke="currentColor" fill="none" x="269.6" y="42" width="72.8" height="30" rx="5"></rect><text fill="currentColor" x="306" y="61" text-anchor="middle" opacity="1">Members</text></g> <g><path stroke="currentColor" fill="none" d="M 516 113 C 582 113, 572 135, 620 135" marker-end="url(#alc-q-arrow-rl56lf)"></path><rect stroke="currentColor" fill="none" x="443.2" y="98" width="72.8" height="30" rx="5"></rect><text fill="currentColor" x="479.59999999999997" y="117" text-anchor="middle" opacity="0.75">Join kim</text></g> <g><rect stroke="currentColor" fill="none" x="622" y="30" width="113.8" height="46" rx="8"></rect><text fill="currentColor" x="678.9" y="50" text-anchor="middle">lobby</text> <text fill="currentColor" x="678.9" y="66" text-anchor="middle">Room instance</text></g> <g><rect stroke="currentColor" fill="none" x="622" y="112" width="113.8" height="46" rx="8"></rect><text fill="currentColor" x="678.9" y="132" text-anchor="middle">kitchen</text> <text fill="currentColor" x="678.9" y="148" text-anchor="middle">Room instance</text></g></svg> <svg viewBox="0 0 318 294" style="max-width: 366px" role="img" aria-label="The lobby handles Join sam while Join ada and Members wait behind it. The kitchen handles its own message at the same time."><defs><marker id="alc-q-arrow-n-rl56lf" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="109" y="0" width="100" height="46" rx="8"></rect><text fill="currentColor" x="159" y="27" text-anchor="middle">callers</text> <line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="46" x2="159" y2="68" marker-end="url(#alc-q-arrow-n-rl56lf)"></line><text fill="currentColor" x="167" y="64">getByName</text></g> <text fill="currentColor" x="4" y="82">Room · two instances</text> <rect stroke="currentColor" fill="none" x="0" y="90" width="318" height="96" rx="10"></rect><text fill="currentColor" x="12" y="121">lobby</text> <text fill="currentColor" x="12" y="163">kitchen</text> <path stroke="currentColor" fill="none" d="M 269.59999999999997 132 V 192 C 269.59999999999997 216, 97.1 206, 97.1 240" marker-end="url(#alc-q-arrow-n-rl56lf)"></path><path stroke="currentColor" fill="none" d="M 269.59999999999997 174 V 192 C 269.59999999999997 216, 220.9 206, 220.9 240" marker-end="url(#alc-q-arrow-n-rl56lf)"></path><g><rect stroke="currentColor" x="233.2" y="102" width="72.8" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="269.59999999999997" y="121" text-anchor="middle" opacity="0.75">Join sam</text></g> <g><rect stroke="currentColor" fill="none" x="150.4" y="102" width="72.8" height="30" rx="5"></rect><text fill="currentColor" x="186.8" y="121" text-anchor="middle" opacity="1">Join ada</text></g> <g><rect stroke="currentColor" x="233.2" y="144" width="72.8" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="269.59999999999997" y="163" text-anchor="middle" opacity="0.75">Join kim</text></g> <g><rect stroke="currentColor" fill="none" x="40.2" y="242" width="113.8" height="46" rx="8"></rect><text fill="currentColor" x="97.1" y="262" text-anchor="middle">lobby</text> <text fill="currentColor" x="97.1" y="278" text-anchor="middle">Room instance</text></g> <g><rect stroke="currentColor" fill="none" x="164" y="242" width="113.8" height="46" rx="8"></rect><text fill="currentColor" x="220.9" y="262" text-anchor="middle">kitchen</text> <text fill="currentColor" x="220.9" y="278" text-anchor="middle">Room instance</text></g></svg>

The lobby handles Join sam while Join ada and Members wait behind it. The kitchen handles its own message at the same time.

## Declare and implement it

An entity is declared from an Effect Cluster entity, made with Effect’s `Entity.make` from a type and the messages it accepts. The class takes it with the entity’s implementation:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import { Entity } from "effect/cluster";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";
import * as Ref from "effect/Ref";
import { Rpc } from "effect/rpc";
import * as Schema from "effect/Schema";
import { Default } from "./Cluster.ts";

export class RoomFull extends Schema.TaggedError<RoomFull>()("RoomFull", {
  capacity: Schema.Int,
}) {}

const Rooms = Entity.make("Room", [
  Rpc.make("Join", { payload: { name: Schema.String }, success: Schema.Int, error: RoomFull }),
  Rpc.make("Members", { success: Schema.Array(Schema.String) }),
]);

export default class Room extends Cluster.Entity<Room>()(
  Rooms,
  { main: import.meta.url },
  Effect.gen(function* () {
    const capacity = yield* Config.Int("ROOM_CAPACITY");
    return Effect.gen(function* () {
      const members = yield* Ref.make(new Set<string>());
      return Rooms.of({
        Join: ({ payload }) =>
          Effect.gen(function* () {
            const all = yield* Ref.get(members);
            if (all.size >= capacity) return yield* new RoomFull({ capacity });
            const next = new Set(all).add(payload.name);
            yield* Ref.set(members, next);
            return next.size;
          }),
        Members: () => Effect.map(Ref.get(members), (all) => [...all]),
      });
    });
  }).pipe(Effect.provide(Default)),
) {}
```

`Rooms.of` types the handlers against the messages: each takes the request envelope, with the payload at `payload`, and returns the RPC’s success, error or stream.

Providing the pool, `Default`, is what deploys it. `yield* Room` returns the entity’s namespace, from any runtime.

`main` is the module whose default export is this class. Each runner in the pool imports it, so export the class as `default`.

## Two phases

The implementation in `src/Room.ts` above is two Effects, one inside the other.

The outer Effect runs once per runner, when it starts. Resolve what every instance shares there: bindings to other entities, secrets, Config values. The inner Effect runs once per instance, on its first message, and holds that instance’s state:

```typescript
Effect.gen(function* () {
  const capacity = yield* Config.Int("ROOM_CAPACITY");
  return Effect.gen(function* () {
    const members = yield* Ref.make(new Set<string>());
    return Rooms.of({
      Join: ({ payload }) =>
        Effect.gen(function* () {
          const all = yield* Ref.get(members);
          if (all.size >= capacity) return yield* new RoomFull({ capacity });
          const next = new Set(all).add(payload.name);
          yield* Ref.set(members, next);
          return next.size;
        }),
      Members: () => Effect.map(Ref.get(members), (all) => [...all]),
    });
  });
});
```

Every room on a runner reads the same `capacity`. Each room gets its own `members`:

<svg viewBox="0 0 647.8 146" style="max-width: 745px" role="img" aria-label="The runner builds the outer Effect once. Each room builds the inner Effect on its first message, then handles every later message with the state it made."><g><text fill="currentColor" x="4" y="34">runner</text> <g><rect stroke="currentColor" fill="none" x="87.00000000000001" y="18" width="67.19999999999997" height="24" rx="5"></rect><text fill="currentColor" x="120.6" y="34" text-anchor="middle">outer</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="647.8" y2="52"></line><text fill="currentColor" x="4" y="78">lobby</text> <g><rect stroke="currentColor" fill="none" x="187.8" y="62" width="44.79999999999998" height="24" rx="5"></rect><text fill="currentColor" x="210.2" y="78" text-anchor="middle">inner</text></g> <g><rect stroke="currentColor" fill="none" x="232.6" y="62" width="78.4" height="24" rx="5"></rect><text fill="currentColor" x="271.8" y="78" text-anchor="middle">Join</text></g> <g><rect stroke="currentColor" fill="none" x="311" y="62" width="78.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="350.2" y="78" text-anchor="middle">Join</text></g> <g><rect stroke="currentColor" fill="none" x="501.40000000000003" y="62" width="100.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="551.8000000000001" y="78" text-anchor="middle">Members</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="647.8" y2="96"></line><text fill="currentColor" x="4" y="122">kitchen</text> <g><rect stroke="currentColor" fill="none" x="355.8" y="106" width="44.799999999999955" height="24" rx="5"></rect><text fill="currentColor" x="378.2" y="122" text-anchor="middle">inner</text></g> <g><rect stroke="currentColor" fill="none" x="400.59999999999997" y="106" width="89.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="445.4" y="122" text-anchor="middle">Join</text></g></g></svg> <svg viewBox="0 0 330 206" style="max-width: 379px" role="img" aria-label="The runner builds the outer Effect once. Each room builds the inner Effect on its first message, then handles every later message with the state it made."><g><text fill="currentColor" x="0" y="22">runner</text> <g><rect stroke="currentColor" fill="none" x="6.36" y="32" width="38.16" height="24" rx="5"></rect><text fill="currentColor" x="50.519999999999996" y="48" text-anchor="start">outer</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">lobby</text> <g><rect stroke="currentColor" fill="none" x="63.6" y="86" width="25.43999999999999" height="24" rx="5"></rect><text fill="currentColor" x="63.6" y="122" text-anchor="start">inner</text></g> <g><rect stroke="currentColor" fill="none" x="89.03999999999999" y="86" width="44.52000000000001" height="24" rx="5"></rect><text fill="currentColor" x="111.3" y="102" text-anchor="middle">Join</text></g> <g><rect stroke="currentColor" fill="none" x="133.56" y="86" width="44.51999999999998" height="24" rx="5"></rect><text fill="currentColor" x="155.82" y="102" text-anchor="middle">Join</text></g> <g><rect stroke="currentColor" fill="none" x="241.68" y="86" width="57.24000000000001" height="24" rx="5"></rect><text fill="currentColor" x="270.3" y="102" text-anchor="middle">Members</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="131" x2="330" y2="131"></line><text fill="currentColor" x="0" y="145">kitchen</text> <g><rect stroke="currentColor" fill="none" x="159" y="155" width="25.439999999999998" height="24" rx="5"></rect><text fill="currentColor" x="159" y="191" text-anchor="start">inner</text></g> <g><rect stroke="currentColor" fill="none" x="184.44" y="155" width="50.879999999999995" height="24" rx="5"></rect><text fill="currentColor" x="209.88" y="171" text-anchor="middle">Join</text></g></g></svg>

phase runsmessage

The runner builds the outer Effect once. Each room builds the inner Effect on its first message, then handles every later message with the state it made.

Inside the inner Effect, `Entity.CurrentAddress`, from `effect/cluster`, holds the instance’s id and shard:

```typescript
const { entityId } = yield* Entity.CurrentAddress;
```

## How long an instance lives

An instance stops after sitting idle for a minute. Its next message starts it again, from the inner Effect, with fresh state:

<svg viewBox="0 0 633 80" style="max-width: 728px" role="img" aria-label="After maxIdleTime with no messages, the lobby stops and its members are gone. The next Join builds the inner Effect again and starts from an empty room."><line stroke-opacity="0.2" stroke="currentColor" x1="374.59999999999997" y1="24" x2="374.59999999999997" y2="74"></line><g><text fill="currentColor" x="4" y="56">lobby</text> <g><rect stroke="currentColor" fill="none" x="72.2" y="40" width="44.8" height="24" rx="5"></rect><text fill="currentColor" x="94.6" y="56" text-anchor="middle">inner</text></g> <g><rect stroke="currentColor" fill="none" x="117" y="40" width="78.4" height="24" rx="5"></rect><text fill="currentColor" x="156.2" y="56" text-anchor="middle">Join</text></g> <g><rect stroke="currentColor" fill="none" x="195.4" y="40" width="179.19999999999996" height="24" rx="5"></rect><text fill="currentColor" x="285" y="56" text-anchor="middle">idle</text></g> <g><rect stroke="currentColor" fill="none" x="453" y="40" width="44.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="475.4" y="56" text-anchor="middle">inner</text></g> <g><rect stroke="currentColor" fill="none" x="497.8" y="40" width="78.39999999999992" height="24" rx="5"></rect><text fill="currentColor" x="537" y="56" text-anchor="middle">Join</text></g></g> <text fill="currentColor" x="374.59999999999997" y="18" text-anchor="middle">maxIdleTime</text></svg> <svg viewBox="0 0 330 105" style="max-width: 379px" role="img" aria-label="After maxIdleTime with no messages, the lobby stops and its members are gone. The next Join builds the inner Effect again and starts from an empty room."><line stroke-opacity="0.2" stroke="currentColor" x1="178.07999999999998" y1="24" x2="178.07999999999998" y2="99"></line><g><text fill="currentColor" x="0" y="44">lobby</text> <g><rect stroke="currentColor" fill="none" x="6.36" y="54" width="25.44" height="24" rx="5"></rect><text fill="currentColor" x="6.36" y="90" text-anchor="start">inner</text></g> <g><rect stroke="currentColor" fill="none" x="31.8" y="54" width="44.519999999999996" height="24" rx="5"></rect><text fill="currentColor" x="54.06" y="70" text-anchor="middle">Join</text></g> <g><rect stroke="currentColor" fill="none" x="76.32" y="54" width="101.75999999999999" height="24" rx="5"></rect><text fill="currentColor" x="127.19999999999999" y="70" text-anchor="middle">idle</text></g> <g><rect stroke="currentColor" fill="none" x="222.6" y="54" width="25.440000000000026" height="24" rx="5"></rect><text fill="currentColor" x="222.6" y="90" text-anchor="start">inner</text></g> <g><rect stroke="currentColor" fill="none" x="248.04000000000002" y="54" width="44.51999999999998" height="24" rx="5"></rect><text fill="currentColor" x="270.3" y="70" text-anchor="middle">Join</text></g></g> <text fill="currentColor" x="178.07999999999998" y="18" text-anchor="middle">maxIdleTime</text></svg>

After maxIdleTime with no messages, the lobby stops and its members are gone. The next Join builds the inner Effect again and starts from an empty room.

Set how long it waits, and how many of its messages may run at once, next to `main`:

```typescript
{ main: import.meta.url, maxIdleTime: "10 minutes", concurrency: 1 }
```

`mailboxCapacity` sets how many messages an instance may hold before new ones fail with `MailboxFull`. A persisted message doesn’t fail: it is stored, and runs once there is room. `disableFatalDefects` sends a handler’s defect to its caller instead of restarting the instance.

An instance also stops when its runner does, including on every deploy, and when its shard moves to another runner. Keep state that must outlive it in a database.

## Keep its code out of its callers

A caller imports the class to call it, and the class carries its implementation with it: the handlers, everything they import, and the pool Layer.

Inside a pool that costs nothing. The pool already bundles every class it hosts into one program, so an entity calling another on the same pool adds no code.

Outside the pool it does. A Worker, a Lambda or a container that calls `Room` would bundle the room’s code and its pool, which it never runs. So would an entity on a different pool, whose own runners never host a room.

For those callers, declare the entity on its own, from its type and its messages, the same arguments Effect’s `Entity.make` takes:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import { Rpc } from "effect/rpc";
import * as Schema from "effect/Schema";

export class Room extends Cluster.Entity<Room>()("Room", [
  Rpc.make("Join", { payload: { name: Schema.String }, success: Schema.Int }),
  Rpc.make("Members", { success: Schema.Array(Schema.String) }),
]) {}
```

Callers import this declaration. It holds the schemas they need to call the room, and nothing else.

The implementation moves to a file of its own, as a Layer made with `.make`:

```typescript
import * as Effect from "effect/Effect";
import * as Ref from "effect/Ref";
import { Default } from "./Cluster.ts";
import { Room } from "./Room.ts";

export default Room.make(
  { main: import.meta.url },
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

`main` is now this file, and its default export is the Layer. Provide the Layer to the Stack, as [Pools](pools.md#deploying-with-layers) shows.

## Messages

Each RPC’s schema shapes what callers send and get back. Effect Cluster’s own annotations, in `ClusterSchema`, and its helpers work unchanged:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import { ClusterSchema } from "effect/cluster";
import { Rpc } from "effect/rpc";
import * as Schema from "effect/Schema";

export class Room extends Cluster.Entity<Room>()("Room", [
  // Written to the database first, delivered at least once, handled once per key.
  Rpc.make("Post", {
    payload: { messageId: Schema.String, text: Schema.String },
    primaryKey: ({ messageId }) => messageId,
  }).annotate(ClusterSchema.Persisted, true),
  // A stream of values back to the caller.
  Rpc.make("History", { success: Schema.String, stream: true }),
]) {}
```

In a handler, `Rpc.fork` lets a message run beside the others, and `Entity.keepAlive` keeps the instance from going idle.

## Use an Effect Cluster entity you already have

A bare declaration also takes an entity made with Effect’s `Entity.make`, as the class with its implementation does, and exposes it as `Room.entity`:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import { Room as RoomEntity } from "./entities/room.ts";

export class Room extends Cluster.Entity<Room>()(RoomEntity) {}
```

To deploy a whole Effect Cluster program as it is, see [Existing programs](existing-programs.md).
