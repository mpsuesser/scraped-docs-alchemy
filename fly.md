---
url: https://alchemy.run/fly
title: "Fly"
description: "Run Effect programs on Fly.io with Alchemy. Declare Services, Machines, Volumes, Postgres, Redis and Tigris in TypeScript, in the regions near your users."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Fly.io is a public cloud that runs your apps as fast-booting virtual machines, called Machines, in dozens of regions close to your users. A built-in proxy sends each request to the nearest Machine. Every app in an organization shares a private network over WireGuard, which also reaches your laptop or another cloud. Postgres, Redis and object storage sit on the same network.

**New to Fly?** Start with [What is Fly?](fly/what-is-fly.md).

## A quick example

A Service is your program on Machines, in an App of its own:

```typescript
import * as Fly from "@alchemy.run/fly";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends Fly.Service<Api>()(
  "Api",
  { main: import.meta.url, region: ["iad", "lhr"] },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello from Fly")),
    };
  }),
) {}
```

And the Stack deploys it to your organization, with a public `{app}.fly.dev` URL:

```typescript
import * as Alchemy from "alchemy";
import * as Fly from "@alchemy.run/fly";
import * as Effect from "effect/Effect";
import Api from "./src/api.ts";

export default Alchemy.Stack(
  "MyApp",
  { providers: Fly.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.url };
  }),
);
```

## Where to start

**Want to build?** The [tutorial](fly/tutorial/part-1.md) goes from an empty directory to a public Service with a Volume and a secret, one step at a time. Start with [Setup](fly/setup.md) to connect your organization.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).
