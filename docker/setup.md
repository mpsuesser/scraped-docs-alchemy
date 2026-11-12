---
url: https://alchemy.run/docker/setup
title: "Setup"
description: "Install @alchemy.run/docker, register its providers, and point Alchemy at a Docker or Podman engine."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Prerequisites

- [Node.js](https://nodejs.org/) 24.11.1+ or [Bun](https://bun.sh/)
- A Docker engine the `docker` CLI can reach: [Docker Desktop](https://docs.docker.com/desktop/), [Docker Engine](https://docs.docker.com/engine/install/) on Linux, a remote host, or [Podman](https://podman.io/)

Check that the CLI reaches an engine:

```sh
docker version
```

## Install

```sh
pnpm add alchemy@latest @alchemy.run/docker@latest effect
```

```typescript
import * as Alchemy from "alchemy";
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Docker.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const web = yield* Docker.Container("web", {
      image: "nginx:alpine",
      ports: [{ external: 8080, internal: 80 }],
      start: true,
    });
    return { ports: web.ports };
  }),
);
```

There’s nothing to sign in to. Alchemy runs the `docker` CLI, and the CLI’s current context decides which engine it talks to. When Docker runs next to a cloud provider, merge the two: `providers: Layer.merge(Cloudflare.providers(), Docker.providers())`.

```sh
pnpm alchemy deploy
```

## Use Podman or another binary

Pass the CLI to run:

```typescript
providers: Docker.providers({ bin: "podman" }),
```

The `DOCKER_BIN` environment variable overrides `bin` on one machine, without changing the code:

```sh
DOCKER_BIN=/opt/homebrew/bin/docker pnpm alchemy deploy
```

## In CI

A CI runner needs the `docker` CLI and an engine, and nothing else: no profile, no token. GitHub’s hosted Linux runners have both. To deploy to a remote engine from CI, give the job the SSH key for that host and declare a [`Docker.Context`](engines-and-contexts.md) for it. Registry passwords come from `Config`, like any other secret. See [Registries & credentials](registries.md) and [CI](../environments/ci.md).

Next, choose where your containers run in [Engines & contexts](engines-and-contexts.md).
