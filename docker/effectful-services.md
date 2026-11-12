---
url: https://alchemy.run/docker/effectful-services
title: "Effectful services"
description: "Write a Swarm service as an Effect program in your own code — Alchemy bundles it into an image on the swarm's engine, serves its fetch handler, runs its background loops and gives it the Config it reads."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A `Docker.Service` can run your own program instead of a pre-built image. Write an Effect that returns a `fetch` handler, and Alchemy bundles that module and its imports into one file, builds an image on the swarm’s engine that runs it with [Bun](https://bun.sh/), and deploys it as the service. Every replica serves HTTP on `port` and runs any background loops you start. No Dockerfile and no registry are needed while the swarm has one node.

src/api.tsmain · your program

Api imagebuilt on the engine

swarmreplicas: 2

Api.1task ·:3000

Api.2task ·:3000

The program is bundled into an image named after its content, so only a code change builds a new one. Each replica runs the same image.

## Write the service

Declare the service in its own module, which is also its `main`:

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export const Swarm = Docker.Swarm("swarm");

export default class Api extends Docker.Service<Api>()(
  "Api",
  Effect.gen(function* () {
    const swarm = yield* Swarm;
    return {
      context: swarm,
      main: import.meta.url,
      port: 3000,
      ports: [{ external: 8080, internal: 3000 }],
      replicas: 2,
    };
  }),
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello from the swarm")),
    };
  }),
) {}
```

Then deploy it from the Stack:

```typescript
import * as Alchemy from "alchemy";
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import Api from "./src/api.ts";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Docker.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const api = yield* Api;
    return { service: api.name };
  }),
);
```

`port` is where the program listens inside each task, and `ports` publishes it on the swarm, here at `localhost:8080`. The service takes every other [Service](services.md) prop too: rollouts, networks, secrets and placement work the same.

## Run background work

`ServerHost.run` starts a loop that runs next to the HTTP server, in every replica:

```typescript
import * as Docker from "@alchemy.run/docker";
import { ServerHost } from "alchemy/Runtime/Process";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import * as Schedule from "effect/Schedule";

export const Swarm = Docker.Swarm("swarm");

export default class Api extends Docker.Service<Api>()(
  "Api",
  Effect.gen(function* () {
    const swarm = yield* Swarm;
    return { context: swarm, main: import.meta.url, port: 3000, replicas: 2 };
  }),
  Effect.gen(function* () {
    const host = yield* ServerHost;
    yield* host.run(
      Effect.log("heartbeat").pipe(
        Effect.repeat(Schedule.spaced("30 seconds")),
        Effect.asVoid,
      ),
    );
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello from the swarm")),
    };
  }),
) {}
```

With two replicas, the loop runs twice, once in each task.

## Read configuration

Read `Config` while the service is constructed, and the value you deploy with is given to every task:

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Config from "effect/Config";

  Effect.gen(function* () {
    const greeting = yield* Config.String("GREETING");
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello from the swarm")),
      fetch: Effect.succeed(HttpServerResponse.text(greeting)),
    };
  }),
```

The value travels in the service’s environment, where `docker service inspect` can read it. See [Secrets](../environments/secrets.md) for how `Config` values are bound, and [Mount secrets and configs](services.md#mount-secrets-and-configs) for passwords on a shared swarm.

## When it rebuilds

The image is tagged with a hash of the bundle and its generated Dockerfile. A deploy with no code change builds nothing and leaves the service alone. A change builds a new image and rolls the tasks over to it.

`image` sets the base image, `oven/bun:1` by default. Any base works as long as it can run Bun.

## More than one node

The image is built on the engine of the service’s `context`, and exists only there. That’s enough for a one-node swarm. Every node of a larger swarm needs to pull the image, so build it with [`Docker.Image`](build-images.md), push it to a [registry](registries.md), and pass that as `image` instead of `main`.
