---
url: https://alchemy.run/docker/build-caches
title: "Build caches"
description: "Keep an image's build layers in a registry or GitHub Actions cache, so fresh CI runners skip the steps that didn't change."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A build reuses the layers it finds in its **builder’s cache**, and the builder lives on one machine. Your laptop keeps its layers between builds. A CI runner starts empty every time, so without help it reruns every step of the Dockerfile on every deploy. A **remote cache** stores the layers somewhere every builder can read them, such as the registry next to the image.

<svg viewBox="0 0 707 164" style="max-width: 813px" role="img" aria-label="The same change to src, built three ways. A fresh runner reruns the install. With cache: true it pulls the unchanged layers from the registry, builds the one step that changed, and saves the layers for the next run."><g><text fill="currentColor" x="4" y="28">your laptop</text> <text fill="currentColor" x="4" y="42">second build</text> <g><rect stroke="currentColor" fill="none" x="135" y="18" width="78.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="174.2" y="34" text-anchor="middle">reused</text></g> <g><rect stroke="currentColor" fill="none" x="213.39999999999998" y="18" width="67.20000000000005" height="24" rx="5"></rect><text fill="currentColor" x="247" y="34" text-anchor="middle">COPY src</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="707" y2="52"></line><text fill="currentColor" x="4" y="72">fresh CI runner</text> <text fill="currentColor" x="4" y="86">no cache</text> <g><rect stroke="currentColor" fill="none" x="135" y="62" width="89.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="179.8" y="78" text-anchor="middle">FROM</text></g> <g><rect stroke="currentColor" fill="none" x="224.60000000000002" y="62" width="369.6" height="24" rx="5"></rect><text fill="currentColor" x="409.40000000000003" y="78" text-anchor="middle">RUN bun install</text></g> <g><rect stroke="currentColor" fill="none" x="594.2" y="62" width="67.19999999999993" height="24" rx="5"></rect><text fill="currentColor" x="627.8" y="78" text-anchor="middle">COPY src</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="707" y2="96"></line><text fill="currentColor" x="4" y="116">fresh CI runner</text> <text fill="currentColor" x="4" y="130">cache: true</text> <g><rect stroke="currentColor" fill="none" x="135" y="106" width="145.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="207.8" y="122" text-anchor="middle">pull:buildcache</text></g> <g><rect stroke="currentColor" fill="none" x="280.6" y="106" width="67.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="314.20000000000005" y="122" text-anchor="middle">COPY src</text></g> <g><rect stroke="currentColor" fill="none" x="347.8" y="106" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="415" y="122" text-anchor="middle">push:buildcache</text></g></g> <line stroke-opacity="0.2" stroke="currentColor" x1="135" y1="140" x2="695" y2="140"></line><text fill="currentColor" x="135" y="156" text-anchor="start">build starts</text></svg> <svg viewBox="0 0 330 239" style="max-width: 379px" role="img" aria-label="The same change to src, built three ways. A fresh runner reruns the install. With cache: true it pulls the unchanged layers from the registry, builds the one step that changed, and saves the layers for the next run."><g><text fill="currentColor" x="0" y="22">your laptop <tspan fill="currentColor" dx="8">second build</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="44.519999999999996" height="24" rx="5"></rect><text fill="currentColor" x="0" y="68" text-anchor="start">reused</text></g> <g><rect stroke="currentColor" fill="none" x="44.519999999999996" y="32" width="38.16000000000001" height="24" rx="5"></rect><text fill="currentColor" x="88.68" y="48" text-anchor="start">COPY src</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="77" x2="330" y2="77"></line><text fill="currentColor" x="0" y="91">fresh CI runner <tspan fill="currentColor" dx="8">no cache</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="101" width="50.88" height="24" rx="5"></rect><text fill="currentColor" x="25.44" y="117" text-anchor="middle">FROM</text></g> <g><rect stroke="currentColor" fill="none" x="50.88" y="101" width="209.88" height="24" rx="5"></rect><text fill="currentColor" x="155.82" y="117" text-anchor="middle">RUN bun install</text></g> <g><rect stroke="currentColor" fill="none" x="260.76" y="101" width="38.160000000000025" height="24" rx="5"></rect><text fill="currentColor" x="260.76" y="137" text-anchor="start">COPY src</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="146" x2="330" y2="146"></line><text fill="currentColor" x="0" y="160">fresh CI runner <tspan fill="currentColor" dx="8">cache: true</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="170" width="82.68" height="24" rx="5"></rect><text fill="currentColor" x="0" y="206" text-anchor="start">pull:buildcache</text></g> <g><rect stroke="currentColor" fill="none" x="82.68" y="170" width="38.16" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="120.84" y="170" width="76.32" height="24" rx="5"></rect><text fill="currentColor" x="203.16" y="186" text-anchor="start">push:buildcache</text></g></g> <line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="215" x2="318" y2="215"></line><text fill="currentColor" x="0" y="231" text-anchor="start">build starts</text></svg>

The same change to src, built three ways. A fresh runner reruns the install. With cache: true it pulls the unchanged layers from the registry, builds the one step that changed, and saves the layers for the next run.

## Cache layers next to the image

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
      build: { context: "./api", cache: true },
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

`cache: true` keeps the layers at the image repository’s `:buildcache` tag, here `ghcr.io/acme/api:buildcache`, using the image’s `registry` credentials. It keeps every stage’s layers, not only the final image’s, so a multi-stage build skips its build stage too.

## Use another cache backend

For any other [BuildKit cache backend](https://docs.docker.com/build/cache/backends/), pass its `--cache-from` and `--cache-to` values. In GitHub Actions:

```typescript
const image = yield* Docker.Image("api", {
  name: "acme/api",
  build: {
    context: "./api",
    cache: {
      from: ["type=gha,scope=api"],
      to: ["type=gha,scope=api,mode=max"],
    },
  },
});
```

Backends like `gha` and `s3` take their credentials from the environment. These values appear on the `docker` command line, so never put a secret in them.

## The builder must support it

Saving a cache needs a builder that can export one: a `docker-container` Buildx builder, which `docker/setup-buildx-action` creates in GitHub Actions, or the default builder with the containerd image store turned on.

The cache never changes the image itself, so turning it on or off doesn’t rebuild or redeploy anything.

## The same option elsewhere

Every resource that builds an image takes the same `cache` option: [Cloudflare Containers](../cloudflare/compute/containers.md), [ECS](../aws/compute/ecs.md), [Cloud Run](../gcp/compute/cloud-run.md) and [Kubernetes workloads](../kubernetes/workloads/images.md) under `build`, and `AWS.ECR.Image` and Lambda container images next to `buildArgs`.
