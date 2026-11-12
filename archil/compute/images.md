---
url: https://alchemy.run/archil/compute/images
title: "Images"
description: "Boot sandboxes from public images, private registries, or a Dockerfile built on your machine and pushed for Archil to pull."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A sandbox boots an OCI image for `linux/amd64`. Archil pulls the image once, records the digest it resolved, and every sandbox booted from it starts from that digest. Archil has no builder: it pulls images, so an image from a Dockerfile is built on your machine and pushed to a registry first.

## Archil pulls, you build

<svg viewBox="0 0 715 352" style="max-width: 822px" role="img" aria-label="Alchemy builds the Dockerfile with your Docker and pushes it to your registry. Archil pulls it from there and boots sandboxes from the digest it pulled."><defs><marker id="alc-dag-arrow-sc35az" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 193 56 L 267 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-sc35az)"></path></g><g><path d="M 357.5 88 L 357.5 138" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-sc35az)"></path><text fill="currentColor" x="365.5" y="109" text-anchor="start">push</text></g> <g><path d="M 442 176 L 516 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-sc35az)"></path><text fill="currentColor" x="479" y="170" text-anchor="middle">pull</text></g> <g><path d="M 606.5 208 L 606.5 258" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-sc35az)"></path><text fill="currentColor" x="614.5" y="229" text-anchor="start">boots</text></g> <g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="169" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="84.5" y="28" text-anchor="middle" dominant-baseline="middle">Dockerfile</text><text fill="currentColor" x="84.5" y="46" text-anchor="middle" dominant-baseline="middle">./agent</text></g> <g transform="translate(273, 24)"><rect stroke="currentColor" fill="none" width="169" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="84.5" y="28" text-anchor="middle" dominant-baseline="middle">docker build</text> <text fill="currentColor" x="84.5" y="46" text-anchor="middle" dominant-baseline="middle">your machine</text></g> <g transform="translate(273, 144)"><rect stroke="currentColor" fill="none" width="169" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="84.5" y="28" text-anchor="middle" dominant-baseline="middle">ghcr.io/acme/agent</text> <text fill="currentColor" x="84.5" y="46" text-anchor="middle" dominant-baseline="middle">your registry</text></g> <g transform="translate(522, 144)"><rect stroke="currentColor" fill="none" width="169" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="84.5" y="28" text-anchor="middle" dominant-baseline="middle">MyImage</text> <text fill="currentColor" x="84.5" y="46" text-anchor="middle" dominant-baseline="middle">Archil.Image · digest</text></g> <g transform="translate(522, 264)"><rect stroke="currentColor" fill="none" width="169" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="84.5" y="28" text-anchor="middle" dominant-baseline="middle">task-1</text> <text fill="currentColor" x="84.5" y="46" text-anchor="middle" dominant-baseline="middle">sandbox</text></g></svg> <svg viewBox="0 0 181 580" style="max-width: 208px" role="img" aria-label="Alchemy builds the Dockerfile with your Docker and pushes it to your registry. Archil pulls it from there and boots sandboxes from the digest it pulled."><defs><marker id="alc-dag-arrow-nc394v" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 90.5 74 C 90.5 101, 90.5 101, 90.5 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-nc394v)"></path></g><g><path d="M 90.5 198 C 90.5 225, 90.5 225, 90.5 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-nc394v)"></path><text fill="currentColor" x="90.5" y="219" text-anchor="middle">push</text></g> <g><path d="M 90.5 322 C 90.5 349, 90.5 349, 90.5 376" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-nc394v)"></path><text fill="currentColor" x="90.5" y="343" text-anchor="middle">pull</text></g> <g><path d="M 90.5 446 C 90.5 473, 90.5 473, 90.5 500" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-nc394v)"></path><text fill="currentColor" x="90.5" y="467" text-anchor="middle">boots</text></g> <g transform="translate(10, 10)"><rect stroke="currentColor" fill="none" width="161" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="80.5" y="28" text-anchor="middle" dominant-baseline="middle">Dockerfile</text><text fill="currentColor" x="80.5" y="46" text-anchor="middle" dominant-baseline="middle">./agent</text></g> <g transform="translate(10, 134)"><rect stroke="currentColor" fill="none" width="161" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="80.5" y="28" text-anchor="middle" dominant-baseline="middle">docker build</text> <text fill="currentColor" x="80.5" y="46" text-anchor="middle" dominant-baseline="middle">your machine</text></g> <g transform="translate(10, 258)"><rect stroke="currentColor" fill="none" width="161" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="80.5" y="28" text-anchor="middle" dominant-baseline="middle">ghcr.io/acme/agent</text> <text fill="currentColor" x="80.5" y="46" text-anchor="middle" dominant-baseline="middle">your registry</text></g> <g transform="translate(10, 382)"><rect stroke="currentColor" fill="none" width="161" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="80.5" y="28" text-anchor="middle" dominant-baseline="middle">MyImage</text> <text fill="currentColor" x="80.5" y="46" text-anchor="middle" dominant-baseline="middle">Archil.Image · digest</text></g> <g transform="translate(10, 506)"><rect stroke="currentColor" fill="none" width="161" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="80.5" y="28" text-anchor="middle" dominant-baseline="middle">task-1</text> <text fill="currentColor" x="80.5" y="46" text-anchor="middle" dominant-baseline="middle">sandbox</text></g></svg>

Alchemy builds the Dockerfile with your Docker and pushes it to your registry. Archil pulls it from there and boots sandboxes from the digest it pulled.

## Boot a public image directly

A public image needs no resource. Name it as a sandbox’s `baseImage`:

```typescript
export const DevBox = Archil.Sandbox("DevBox", { baseImage: "python:3.13" });
```

A sandbox with neither `baseImage` nor `image` boots `node:24-bookworm`.

## Pin an image by its digest

`Archil.Image` pulls an image once and keeps its digest, so every sandbox boots the same bits even after the tag moves:

```typescript
export const Base = Archil.Image("Base", { image: "node:24-bookworm" });
```

Pass its `imageId` to a sandbox:

```typescript
export const DevBox = Effect.gen(function* () {
  const base = yield* Base;
  return yield* Archil.Sandbox("DevBox", { image: base.imageId });
});
```

Changing `image` pulls it again.

## Pull from a private registry

Give the registry’s credentials. Archil uses them to pull this image and doesn’t store them:

```typescript
export const MyImage = Archil.Image("MyImage", {
  image: "ghcr.io/acme/agent:1",
  registry: {
    server: "ghcr.io",
    username: "acme-bot",
    password: Config.Redacted("GHCR_TOKEN"),
  },
});
```

## Build a Dockerfile on your machine

Give a `dockerfile` and where to `push` it. Alchemy builds it with your local Docker, pushes it tagged with the build’s hash, and Archil pulls it:

```typescript
export const MyImage = Archil.Image("MyImage", {
  dockerfile: "./agent/Dockerfile",
  context: "./agent",
  push: {
    repository: "ghcr.io/acme/agent",
    credentials: { server: "ghcr.io", username: "acme-bot", password: Config.Redacted("GHCR_TOKEN") },
  },
});
```

The credentials need to push to the repository and pull from it. A change to the Dockerfile or its context builds and pushes a new tag.

## Pull images while your app runs

`Archil.CreateImage` pulls images at run time, e.g. one per customer repository, with the same props as `Archil.Image`:

```typescript
const images = yield* Archil.CreateImage();
const image = yield* images.build({ image: \`ghcr.io/acme/${tenant}:latest\` });
const sandbox = yield* sandboxes.create({ name: tenant, image: image.imageId });
```

## Programs that need a filesystem client

A [program sandbox](programs.md) boots a public image. When one of its bindings installs a filesystem client, like a mount for another storage service, Alchemy builds an image of the base plus that client the same way, so give the sandbox a `push` too.

## Images stay in your registry

Archil has no API to delete an image. Destroying an `Archil.Image` removes it from Alchemy’s state only, and the image pushed to your registry stays there.
