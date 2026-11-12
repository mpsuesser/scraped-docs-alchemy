---
url: https://alchemy.run/docker
title: "Docker"
description: "Docker builds applications into images and runs them as isolated containers on a Docker engine, on your laptop, a CI runner or a server."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Docker builds an application into an image, a layered filesystem snapshot plus the command to run, and runs images as isolated containers on a Docker engine, on your laptop, a CI runner or a server. Registries carry images between machines, which is why nearly every cloud runs containers from them. Swarm, built into the engine, runs replicated services across a cluster of engines.

**New to Docker?** Start with [What is Docker?](docker/what-is-docker.md).

## A quick example

Pull Postgres and run it with its data on a volume:

```typescript
import * as Alchemy from "alchemy";
import * as Docker from "@alchemy.run/docker";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Docker.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const data = yield* Docker.Volume("postgres-data");

    const postgres = yield* Docker.Container("postgres", {
      image: "postgres:18-alpine",
      environment: { POSTGRES_PASSWORD: yield* Config.Redacted("POSTGRES_PASSWORD") },
      ports: [{ external: 15432, internal: 5432 }],
      volumes: [{ hostPath: data.name, containerPath: "/var/lib/postgresql/data" }],
      start: true,
    });

    return { ports: postgres.ports };
  }),
);
```

Build your own image from a Dockerfile, and push it to a registry:

```typescript
const image = yield* Docker.Image("api", {
  name: "acme/api",
  build: { context: "./api" },
  registry: {
    server: "ghcr.io",
    username: "acme",
    password: yield* Config.Redacted("GITHUB_TOKEN"),
  },
});
```

Deploy, and Alchemy creates everything on the engine your `docker` CLI talks to:

```sh
pnpm alchemy deploy
```

## Where to start

**Want to build?** Install the provider in [Setup](docker/setup.md), then follow [Local Postgres](docker/local-services.md) step by step.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Engines & access

- [Engines & contexts](docker/engines-and-contexts.md)
- [Registries & credentials](docker/registries.md)

### Swarm

- [Swarms](docker/swarm.md)
- [Services](docker/services.md)
- [Effectful services](docker/effectful-services.md)

### Guides

- [Local Postgres](docker/local-services.md)
