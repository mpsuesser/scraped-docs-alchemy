---
url: https://alchemy.run/cluster/calling
title: "Calling an entity"
description: "Call an Effect Cluster entity from any runtime by id — RPCs, streams, typed errors, fire-and-forget messages and headers."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

Any runtime calls an entity the same way, a Cloudflare Worker, a Lambda function, a container or another entity: yield the class, then name an instance. The callers below run outside the pool, so they import the room’s declaration on its own, without its code. See [Keep its code out of its callers](entities.md#keep-its-code-out-of-its-callers).

## Get a client

`yield* Room` returns the entity’s namespace. `getByName` returns a client for one instance, with one method per RPC:

```typescript
import * as Lambda from "@alchemy.run/aws/Lambda";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Room } from "./Room.ts";

export default class Api extends Lambda.Function<Api>()(
  "Api",
  { main: import.meta.url, functionUrl: true },
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

The client has the same shape as Effect Cluster’s own `Entity.client`, and the same shape as a Cloudflare Durable Object’s `getByName`.

## How the call travels

While deploying, `yield* Room` binds three values into the caller’s environment: the address of the pool’s gateway, the token it checks, and the pool’s shard group. At runtime, it reads them back and calls the gateway over HTTP. The caller needs no access to the database and no route to the pool.

Inside the cluster, one entity calling another (or a workflow calling an entity) goes straight from runner to runner, through Effect Cluster itself.

<svg viewBox="0 0 442.79999999999995 362" style="max-width: 509px" role="img" aria-label="The Api reaches the pool only through its gateway. An Invite run, inside the pool, calls the lobby straight from its runner."><defs><marker id="alc-bd-arrow-6cclbl" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="118" width="410.79999999999995" height="228" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="334.09999999999997" y1="64" x2="334.09999999999997" y2="144" marker-end="url(#alc-bd-arrow-6cclbl)"></line><text fill="currentColor" x="344.09999999999997" y="110">Join</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="334.09999999999997" y1="196" x2="334.09999999999997" y2="276" marker-end="url(#alc-bd-arrow-6cclbl)"></line></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="183.39999999999998" y1="304" x2="255.39999999999998" y2="304" marker-end="url(#alc-bd-arrow-6cclbl)"></line><text fill="currentColor" x="221.39999999999998" y="296" text-anchor="middle">Join</text></g> <text fill="currentColor" x="28" y="137" text-anchor="start">Default <tspan fill="currentColor" dx="8">pool</tspan></text> <g><rect stroke="currentColor" fill="none" x="259.4" y="16" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="334.09999999999997" y="37" text-anchor="middle">Api</text> <text fill="currentColor" x="334.09999999999997" y="53" text-anchor="middle">Lambda Function</text></g> <g><rect stroke="currentColor" fill="none" x="259.4" y="148" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="334.09999999999997" y="169" text-anchor="middle">gateway</text> <text fill="currentColor" x="334.09999999999997" y="185" text-anchor="middle">HTTP · token</text></g> <g><rect stroke="currentColor" fill="none" x="259.4" y="280" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="334.09999999999997" y="301" text-anchor="middle">lobby</text> <text fill="currentColor" x="334.09999999999997" y="317" text-anchor="middle">Room · Runner 1</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="280" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="301" text-anchor="middle">Invite</text> <text fill="currentColor" x="108.69999999999999" y="317" text-anchor="middle">workflow · Runner 2</text></g></svg> <svg viewBox="0 0 217.39999999999998 580" style="max-width: 250px" role="img" aria-label="The Api reaches the pool only through its gateway. An Invite run, inside the pool, calls the lobby straight from its runner."><defs><marker id="alc-bd-arrow-pfxehy" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="185.39999999999998" height="540" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="250" x2="108.69999999999999" y2="346" marker-end="url(#alc-bd-arrow-pfxehy)"></line><text fill="currentColor" x="118.69999999999999" y="304">Join</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="398" x2="108.69999999999999" y2="494" marker-end="url(#alc-bd-arrow-pfxehy)"></line></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="102" x2="108.69999999999999" y2="494" marker-end="url(#alc-bd-arrow-pfxehy)"></line><text fill="currentColor" x="118.69999999999999" y="304">Join</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">Default <tspan fill="currentColor" dx="8">pool</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="223" text-anchor="middle">Api</text> <text fill="currentColor" x="108.69999999999999" y="239" text-anchor="middle">Lambda Function</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="371" text-anchor="middle">gateway</text> <text fill="currentColor" x="108.69999999999999" y="387" text-anchor="middle">HTTP · token</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="498" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="519" text-anchor="middle">lobby</text> <text fill="currentColor" x="108.69999999999999" y="535" text-anchor="middle">Room · Runner 1</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="75" text-anchor="middle">Invite</text> <text fill="currentColor" x="108.69999999999999" y="91" text-anchor="middle">workflow · Runner 2</text></g></svg>

The Api reaches the pool only through its gateway. An Invite run, inside the pool, calls the lobby straight from its runner.

## The gateway token

Each pool’s gateway checks a token on every request, except `/health`, which load balancers use. The Stack creates one random token per pool and keeps it in its state, and every runtime that yields one of the pool’s classes is given it, so a caller never handles it.

The token stays the same across deploys. Anyone holding it can call every class on that pool, so treat it like any other secret. On Docker, and on Kubernetes unless you set `serviceType`, the gateway is only reachable on the cluster’s own network.

## Typed errors and streams

An RPC’s error comes back with its tag:

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import * as Stream from "effect/Stream";
import { Room } from "./Room.ts";

export default class Api extends Docker.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return {
      fetch: Effect.gen(function* () {
        const lobby = yield* rooms.getByName("lobby");
        const joined = yield* lobby.Join({ name: "sam" }).pipe(
          Effect.catchTag("RoomFull", () => Effect.succeed(-1)),
        );
        const history = yield* Stream.runCollect(lobby.History());
        return yield* HttpServerResponse.json({ joined, history: [...history] });
      }).pipe(Effect.orDie),
    };
  }),
) {}
```

A streaming RPC returns a `Stream` that delivers every value the handler emits.

## Send without waiting

Pass `discard: true` to send a message and return as soon as it’s on its way:

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Room } from "./Room.ts";

export default class Api extends Docker.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return {
      fetch: Effect.gen(function* () {
        const lobby = yield* rooms.getByName("lobby");
        yield* lobby.Join({ name: "sam" }, { discard: true });
        return HttpServerResponse.text("sent");
      }).pipe(Effect.orDie),
    };
  }),
) {}
```

The handler still runs; only the reply is skipped. Errors before the message is sent, like a full mailbox, still fail the call.

## Gateway errors

Besides the RPC’s own errors, a call can fail with Effect Cluster’s: `MailboxFull` when an instance has too many messages waiting and the message isn’t persisted, `AlreadyProcessingMessage` when a persisted message with the same key is still running, `PersistenceError` when the message can’t be stored, and `EntityNotAssignedToRunner` while a shard moves. Each is a tagged error you can retry.

## Headers

Pass `headers` to send request headers along with a message. RPC middleware on the entity reads them; see [RPC middleware](middleware.md).
