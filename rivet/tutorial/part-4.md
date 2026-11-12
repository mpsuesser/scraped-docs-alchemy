---
url: https://alchemy.run/rivet/tutorial/part-4
title: "Part 4: Run It on Hetzner"
description: "Move the chat rooms from Docker to Hetzner Cloud by swapping the control plane, the pool and the API's host."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Nothing in the room mentions Docker. The control plane and the pool are Layers, so you swap them for Hetzner’s, move the Api onto a Hetzner Server, and the same room runs on Hetzner Cloud. This part costs money while it runs; the last step removes it.

## Prerequisites

Finish [Hetzner setup](../../hetzner/setup.md), and install `@alchemy.run/hetzner`.

## Tear down the local rooms

```sh
pnpm alchemy destroy
```

This removes the Docker control plane, the pool and the API. Do it before you swap the providers below: a Stack without `Docker.providers()` can’t delete Docker resources.

## Swap the control plane and the pool

```typescript
import * as RivetDocker from "@alchemy.run/rivet/Docker";
import * as RivetHetzner from "@alchemy.run/rivet/Hetzner";
import * as Layer from "effect/Layer";

export const Cluster = RivetDocker.SwarmControlPlane("Rivet");
export const Cluster = RivetHetzner.ServerControlPlane("Rivet");

export const Default = RivetDocker.SwarmPool("Default").pipe(Layer.provide(Cluster));
export const Default = RivetHetzner.ServerPool("Default").pipe(Layer.provide(Cluster));
```

The control plane runs on a Hetzner Server, behind an HTTPS edge at an `sslip.io` address. The pool gets a Server of its own. `src/room.ts` doesn’t change.

## Give the API a Server

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Rivet from "@alchemy.run/rivet";
import * as Hetzner from "@alchemy.run/hetzner";
import * as Effect from "effect/Effect";
import * as HttpServerRequest from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import * as Schema from "effect/Schema";
import { Cluster } from "./cluster.ts";
import Room from "./room.ts";

const Message = Schema.Struct({ author: Schema.String, text: Schema.String });

export const ApiServer = Hetzner.Server("ApiServer", {
  serverType: "cx23",
  image: "ubuntu-24.04",
  location: "nbg1",
});
```

## Run the API on it

```typescript
export default class Api extends Docker.Service<Api>()(
export default class Api extends Hetzner.Service<Api>()(
  "Api",
  Effect.gen(function* () {
    const cluster = yield* Rivet.Cluster;
    return {
      main: import.meta.url,
      port: 3000,
      ports: [{ external: 3000, internal: 3000 }],
      networks: cluster.network?.kind === "docker" ? [{ name: cluster.network.name }] : [],
    };
  }).pipe(Effect.provide(Cluster)),
  { server: ApiServer, main: import.meta.url },
  Effect.gen(function* () {
    const rooms = yield* Room;
```

The control plane has a public HTTPS address on Hetzner, so the API no longer joins a network. The `fetch` handler stays the same.

## Swap the providers

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Hetzner from "@alchemy.run/hetzner";
import * as Rivet from "@alchemy.run/rivet";

    providers: Layer.mergeAll(Docker.providers(), Rivet.providers()),
    providers: Layer.mergeAll(Hetzner.providers(), Rivet.providers()),

    return { url: "http://localhost:3000", service: api.name };
    return { url: api.url };
```

## Deploy

```sh
pnpm alchemy deploy
```

The rooms start empty: their data was on the Docker control plane. The new control plane takes a few minutes to start.

## Talk in a room

```sh
curl -X POST <url>/lobby/join
# {"visitors":1}
```

Use the `url` the deploy printed. You have the same rooms running on Hetzner, with no change to the room itself.

## Clean up

The Servers and the Load Balancer cost money while they run:

```sh
pnpm alchemy destroy
```
