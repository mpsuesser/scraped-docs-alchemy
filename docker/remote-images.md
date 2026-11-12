---
url: https://alchemy.run/docker/remote-images
title: "Pull & mirror images"
description: "Pull an existing image with Docker.RemoteImage, pin it so deploys stop re-pulling, and re-tag and push it into your own registry."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Most containers run an image someone else built: `postgres`, `nginx`, `redis`. **Pulling** downloads it from a registry into the engine. A **tag** like `postgres:18-alpine` is a moving name that its publisher points at new builds, so two pulls of the same tag a month apart can give different images.

`Docker.RemoteImage` pulls an image into the engine, and can re-tag it and push it to your own registry. A copy in your own registry keeps working when the source changes, rate-limits you, or disappears.

docker.ioregistry

nginx:alpineimage

ghcr.ioyour registry

acme/nginx:alpineimage

your engineDocker.RemoteImage

nginx:alpinepulled

acme/nginx:alpinesame image, new name

A mirror is one image under two names. RemoteImage pulls it, tags the same image under your repository, and pushes that name to your registry.

## Pull an image

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
    const redis = yield* Docker.RemoteImage("redis", { name: "redis", tag: "8-alpine" });

    const cache = yield* Docker.Container("cache", {
      image: redis,
      start: true,
    });

    return { image: redis.imageRef, container: cache.name };
  }),
);
```

Passing the resource as `image` makes the container depend on it. When a pull brings a different image, the container is recreated on it. `platform` pulls for another CPU, like `linux/amd64`.

## Stop re-pulling on every deploy

By default every deploy pulls again, so a moving tag picks up its publisher’s newest build. `alwaysPull: false` keeps whatever the engine already has under that tag, and pulls only when it’s missing:

```typescript
const postgres = yield* Docker.RemoteImage("postgres", {
  name: "postgres",
  tag: "18-alpine",
  alwaysPull: false,
});
```

Deploys then stop touching the image, which is what you want for a local database. To move to a newer build, change the tag.

## Mirror into your registry

`targetName` and `targetTag` re-tag the pulled image, and `registry` pushes it:

```typescript
const mirrored = yield* Docker.RemoteImage("nginx-mirror", {
  name: "nginx",
  tag: "alpine",
  targetName: "acme/nginx",
  targetTag: "alpine",
  registry: {
    server: "ghcr.io",
    username: "acme",
    password: yield* Config.Redacted("GITHUB_TOKEN"),
  },
});
```

`mirrored.imageRef` is `ghcr.io/acme/nginx:alpine`, and `repoDigest` pins its exact content. Hand either to a cloud runtime, as in [Images on cloud runtimes](cloud-runtimes.md).

## Destroying a remote image

Destroying a `RemoteImage` leaves the image on the engine and in your registry. Other stacks, and your own `docker run`, may use the same tag. Remove it yourself with `docker image rm` when you’re done with it.
