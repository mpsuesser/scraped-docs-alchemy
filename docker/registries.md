---
url: https://alchemy.run/docker/registries
title: "Registries & credentials"
description: "Give an image its registry credentials as a Redacted Config value, push without docker login, pull private base images, and read the password from a secrets manager per stage."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **registry** stores images under repositories, like `ghcr.io/acme/api`, and serves them to any engine that pulls. Docker Hub, GitHub’s `ghcr.io`, GitLab and every cloud run one. Public images pull without credentials. Pushing, and pulling a private image, needs a username and a password or token.

Registry credentials are the only credentials Docker resources take. You give them to one resource, and only that resource’s build and push see them. Nothing runs `docker login`, so your own logins on the machine stay as they are.

your machinealchemy deploy

Docker.Image apibuild + push

registry

ghcr.io · GITHUB\_TOKEN

~/.dockeryour logins · untouched

ghcr.ioregistry

acme/apirepository

The token goes to that one Image's build and push, through a config that exists only for the command. Your ~/.docker logins and contexts are never read or changed.

## Push an image

Set `registry` on a `Docker.Image`, and every build is pushed there:

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
    const image = yield* Docker.Image("api", {
      name: "acme/api",
      build: { context: "./api" },
      registry: {
        server: "ghcr.io",
        username: "acme",
        password: yield* Config.Redacted("GITHUB_TOKEN"),
      },
    });

    return { ref: image.imageRef, digest: image.repoDigest };
  }),
);
```

`password` is `Redacted`, so it never prints in logs or plans. After the push, `imageRef` is `ghcr.io/acme/api:latest` and `repoDigest` is `ghcr.io/acme/api@sha256:…`. `skipPush: true` keeps the build local while leaving `registry` in place.

For `ghcr.io`, the password is a GitHub token with the `write:packages` scope. Docker Hub takes a personal access token rather than your account password.

## Pull a private base image

The same credentials authenticate the build, so a Dockerfile can start `FROM` a private image in that registry:

```typescript
// Dockerfile: FROM registry.example.com/base:v1
const image = yield* Docker.Image("api", {
  name: "registry.example.com/acme/api",
  build: { context: "./api" },
  registry: {
    server: "registry.example.com",
    username: "deploy",
    password: yield* Config.Redacted("REGISTRY_PASSWORD"),
  },
});
```

Logging in during a build needs Buildx 0.26+ (Docker Desktop 4.44+). With an older Buildx, the push still works, but a private base image needs a `docker login` on the machine first.

Logins you already pass through `DOCKER_AUTH_CONFIG`, for other registries, are kept alongside these.

## Mirror an image into your registry

`Docker.RemoteImage` takes the same `registry` to push a pulled image under your own name. See [Pull & mirror images](remote-images.md).

## Values from a secrets manager

`.env` is only the default. A Stack’s `secrets` option reads the same `Config` values from a secrets manager instead, like Doppler or Infisical, so a registry token never sits in a file on anyone’s machine. Pick the source per stage:

```typescript
import * as Alchemy from "alchemy";
import * as Docker from "@alchemy.run/docker";
import * as Doppler from "@alchemy.run/doppler";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Docker.providers(),
    state: Alchemy.localState(),
    secrets: ({ stage }) =>
      stage === "prod"
        ? Doppler.Secrets({ project: "my-app", config: "prd" })
        : Alchemy.Secrets.DotEnv(),
  },
  Effect.gen(function* () {
    const image = yield* Docker.Image("api", {
      name: "acme/api",
      build: { context: "./api" },
      registry: {
        server: "ghcr.io",
        username: "acme",
        password: yield* Config.Redacted("GITHUB_TOKEN"),
      },
    });
    return { ref: image.imageRef };
  }),
);
```

Your code doesn’t change: `Config.Redacted("GITHUB_TOKEN")` reads from whichever source the stage uses. See [Secret providers](../environments/secret-providers.md).

## Secrets inside images and containers

A registry token gets an image to the registry. Secrets the build or the app needs are separate:

- A token a `RUN` step needs, like a private npm token, is a build secret. It never lands in the image. See [Build images](build-images.md#use-a-secret-in-a-build).
- A password the app reads while it runs goes in the container’s `environment` as a `Redacted` value. See [Containers](containers.md#pass-secrets-in-the-environment).
- A Swarm service can mount Swarm secrets as files. See [Services](services.md#mount-secrets-and-configs).
