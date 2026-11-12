---
url: https://alchemy.run/cluster/middleware
title: "RPC middleware"
description: "Authenticate and enrich every message to an entity with Effect RPC middleware — a server side that runs on the pool and a client side that runs in the caller."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

RPC middleware wraps every message to an RPC. Its server side runs on the runner, before the handler: it can reject the message, or provide a service the handler uses. Its client side runs in the caller, and usually adds headers.

<svg viewBox="0 0 479.4 268" style="max-width: 551px" role="img" aria-label="The client side runs in the caller and adds the header. The server side runs on the runner, before the handler, and provides CurrentUser."><defs><marker id="alc-bd-arrow-30qzu3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="185.39999999999998" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="278" y="24" width="185.39999999999998" height="228" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="183.39999999999998" y1="78" x2="292" y2="78" marker-end="url(#alc-bd-arrow-30qzu3)"></line><text fill="currentColor" x="239.7" y="70" text-anchor="middle">header</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="370.7" y1="102" x2="370.7" y2="182" marker-end="url(#alc-bd-arrow-30qzu3)"></line><text fill="currentColor" x="380.7" y="148">CurrentUser</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">Api <tspan fill="currentColor" dx="8">caller</tspan></text> <text fill="currentColor" x="290" y="43" text-anchor="start">Default <tspan fill="currentColor" dx="8">pool</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="75" text-anchor="middle">AuthClient</text> <text fill="currentColor" x="108.69999999999999" y="91" text-anchor="middle">sets x-caller-token</text></g> <g><rect stroke="currentColor" fill="none" x="296" y="54" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="370.7" y="75" text-anchor="middle">Auth</text> <text fill="currentColor" x="370.7" y="91" text-anchor="middle">reads the header</text></g> <g><rect stroke="currentColor" fill="none" x="296" y="186" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="370.7" y="207" text-anchor="middle">Room handler</text> <text fill="currentColor" x="370.7" y="223" text-anchor="middle">yield* CurrentUser</text></g></svg> <svg viewBox="0 0 217.39999999999998 432" style="max-width: 250px" role="img" aria-label="The client side runs in the caller and adds the header. The server side runs on the runner, before the handler, and provides CurrentUser."><defs><marker id="alc-bd-arrow-n9ooto" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="185.39999999999998" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="172" width="185.39999999999998" height="244" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="102" x2="108.69999999999999" y2="198" marker-end="url(#alc-bd-arrow-n9ooto)"></line><text fill="currentColor" x="118.69999999999999" y="156">header</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="250" x2="108.69999999999999" y2="346" marker-end="url(#alc-bd-arrow-n9ooto)"></line><text fill="currentColor" x="118.69999999999999" y="304">CurrentUser</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">Api <tspan fill="currentColor" dx="8">caller</tspan></text> <text fill="currentColor" x="28" y="191" text-anchor="start">Default <tspan fill="currentColor" dx="8">pool</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="75" text-anchor="middle">AuthClient</text> <text fill="currentColor" x="108.69999999999999" y="91" text-anchor="middle">sets x-caller-token</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="223" text-anchor="middle">Auth</text> <text fill="currentColor" x="108.69999999999999" y="239" text-anchor="middle">reads the header</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="371" text-anchor="middle">Room handler</text> <text fill="currentColor" x="108.69999999999999" y="387" text-anchor="middle">yield* CurrentUser</text></g></svg>

The client side runs in the caller and adds the header. The server side runs on the runner, before the handler, and provides CurrentUser.

## Declare it

Middleware is Effect’s `RpcMiddleware.Service`. This one provides the current user, and fails with `Unauthorized` without a token:

```typescript
import * as Context from "effect/Context";
import * as Effect from "effect/Effect";
import * as Headers from "effect/http/Headers";
import * as Layer from "effect/Layer";
import { RpcMiddleware } from "effect/rpc";
import * as Schema from "effect/Schema";

export class CurrentUser extends Context.Service<CurrentUser, { readonly id: string }>()(
  "CurrentUser",
) {}

export class CallerToken extends Context.Service<CallerToken, string>()("CallerToken") {}

export class Unauthorized extends Schema.TaggedError<Unauthorized>()("Unauthorized", {}) {}

export class Auth extends RpcMiddleware.Service<Auth, { provides: CurrentUser }>()("Auth", {
  error: Unauthorized,
  requiredForClient: true,
}) {}

/** Runs on the runner, before every handler of an RPC that uses \`Auth\`. */
export const AuthLive = Layer.succeed(Auth, (effect, { headers }) => {
  const token = headers["x-caller-token"];
  return token?.startsWith("user:")
    ? Effect.provideService(effect, CurrentUser, { id: token.slice("user:".length) })
    : Effect.fail(new Unauthorized());
});

/** Runs in the caller: puts its token on each request. */
export const AuthClient = RpcMiddleware.layerClient(Auth, ({ request, next }) =>
  Effect.gen(function* () {
    const token = yield* Effect.serviceOption(CallerToken);
    return yield* next(
      token._tag === "Some"
        ? { ...request, headers: Headers.set(request.headers, "x-caller-token", token.value) }
        : request,
    );
  }),
);
```

## Attach it to RPCs

`.middleware(Auth)` applies it to one RPC:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import { Rpc } from "effect/rpc";
import * as Schema from "effect/Schema";
import { Auth } from "./Auth.ts";

export class Room extends Cluster.Entity<Room>()("Room", [
  Rpc.make("Join", { success: Schema.Int }).middleware(Auth),
  Rpc.make("Members", { success: Schema.Array(Schema.String) }),
]) {}
```

The handler reads what the middleware provides:

```typescript
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import * as Ref from "effect/Ref";
import { AuthLive, CurrentUser } from "./Auth.ts";
import { Default } from "./Cluster.ts";
import { Room } from "./Room.ts";

export default Room.make(
  { main: import.meta.url },
  Effect.gen(function* () {
    return Effect.gen(function* () {
      const members = yield* Ref.make(new Set<string>());
      return Room.of({
        Join: () =>
          Effect.gen(function* () {
            const user = yield* CurrentUser;
            return yield* Ref.modify(members, (all) => {
              const next = new Set(all).add(user.id);
              return [next.size, next];
            });
          }),
        Members: () => Effect.map(Ref.get(members), (all) => [...all]),
      });
    });
  }).pipe(Effect.provide(Default.pipe(Layer.provide(AuthLive)))),
);
```

## Provide the server side with the pool

`AuthLive` goes in the same `Effect.provide` as the pool, provided to it. The pool carries it to the entity on every runner.

Deploying an entity whose RPC uses middleware nobody provided fails the deploy, naming the RPC and the middleware:

```text
Cluster.EntityMiddlewareMissing: Cluster.Entity 'Room' is missing RPC middleware (Join uses Auth).
```

## Provide the client side in the caller

The caller provides `AuthClient` where it makes the call, and whatever the client side reads:

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { AuthClient, CallerToken } from "./Auth.ts";
import { Room } from "./Room.ts";

export default class Api extends Docker.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const lobby = yield* rooms.getByName("lobby");
        const members = yield* lobby.Join().pipe(
          Effect.provideService(CallerToken, request.headers["authorization"] ?? ""),
          Effect.catchTag("Unauthorized", () => Effect.succeed(-1)),
        );
        return yield* HttpServerResponse.json({ members });
      }).pipe(Effect.provide(AuthClient), Effect.orDie),
    };
  }),
) {}
```

The headers it sets reach the entity’s middleware whether the call goes through the gateway or straight from runner to runner.
