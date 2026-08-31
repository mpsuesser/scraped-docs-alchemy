---
url: https://alchemy.run/docker/setup
title: "Setup"
description: "Point alchemy at a Docker daemon — the active CLI context and the DOCKER_BIN override."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
---

Register the provider in your stack:

```typescript
// alchemy.run.ts
import * as Docker from "alchemy/Docker";

providers: Docker.providers(),
state: Alchemy.localState(),
```

That's the whole registration. When Docker runs alongside a cloud
provider, merge the layers:
`providers: Layer.merge(Cloudflare.providers(), Docker.providers())`.

## Prerequisites

A Docker daemon reachable from the `docker` CLI is the only
requirement. The provider shells out through the **active Docker CLI
context**, so whatever `docker ps` talks to — Docker Desktop, a
remote Docker host, an SSH context, a CI runner's daemon — is what
alchemy provisions against. There is no login step and no
stored token: this provider has no credentials of its own.

## Binary override

```sh
DOCKER_BIN=/opt/homebrew/bin/docker bun alchemy deploy
```

The provider resolves the Docker binary from the `DOCKER_BIN`
environment variable, falling back to `docker` on the `PATH`.

## Registry credentials

Registry auth is a per-resource prop, not global configuration.
[`Image`](https://alchemy.run/providers/docker/image) and
[`RemoteImage`](https://alchemy.run/providers/docker/remoteimage) accept a `registry`
with `{ server, username, password }`, where `password` is a
`Redacted` value:

```typescript
const image = yield* Docker.Image("app", {
  name: "my-app",
  build: { context: "./app" },
  registry: {
    server: "ghcr.io",
    username: "octocat",
    password: Config.redacted("GITHUB_TOKEN"),
  },
});
```

Push deliberately skips `docker login`: the credentials are written
into an isolated `DOCKER_CONFIG` that only the push command reads, so
your global docker config, buildx builders, and `docker context`
setup stay intact.

## Next steps

- [Docker overview](../docker.md) — resources and compositions.
- [Run local services](local-services.md) — Postgres with a
  network, volume, and healthcheck.
- [Build and push an image](build-and-push.md) — Dockerfile
  builds and registry push.
