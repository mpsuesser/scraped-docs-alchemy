---
url: https://alchemy.run/hetzner
title: "Hetzner"
description: "Run Effect programs on Hetzner Cloud Servers with Alchemy. Declare Servers, Services, Volumes, networks, firewalls, load balancers and DNS in one TypeScript program."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Hetzner rents dedicated and cloud servers at a fraction of the large clouds’ prices. Hetzner Cloud, its virtual-machine platform, runs in Germany, Finland, the US and Singapore.

Around its Servers, Hetzner Cloud offers just what servers need: network disks, private networks, firewalls, load balancers, IP addresses and DNS. There are no managed functions or queues: you run your own programs on your own machines.

**New to Hetzner?** Start with [What is Hetzner?](hetzner/what-is-hetzner.md).

## A quick example

A Server is a declaration:

```typescript
import * as Hetzner from "@alchemy.run/hetzner";

export const Box = Hetzner.Server("Box", { serverType: "cx22", image: "ubuntu-24.04", location: "nbg1" });
```

A Service is your program on it:

```typescript
import * as Hetzner from "@alchemy.run/hetzner";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Box } from "./server.ts";

export default class Api extends Hetzner.Service<Api>()(
  "Api",
  { server: Box, main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello from Hetzner")),
    };
  }),
) {}
```

And the Stack deploys both to your project:

```typescript
import * as Alchemy from "alchemy";
import * as Hetzner from "@alchemy.run/hetzner";
import * as Effect from "effect/Effect";
import Api from "./src/api.ts";

export default Alchemy.Stack(
  "MyApp",
  { providers: Hetzner.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.url };
  }),
);
```

## Where to start

**Want to build?** The [tutorial](hetzner/tutorial/part-1.md) goes from an empty directory to a Server running an HTTP Service with a Volume and a firewall, one step at a time. Start with [Setup](hetzner/setup.md) to connect your project.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Accounts & access

- [Projects, tokens & SSH keys](hetzner/projects-and-tokens.md)

### Compute

- [Servers](hetzner/compute/servers.md)
- [Services](hetzner/compute/services.md)

### Networking

- [Networking](hetzner/networking.md)
- [DNS](hetzner/networking/dns.md)
