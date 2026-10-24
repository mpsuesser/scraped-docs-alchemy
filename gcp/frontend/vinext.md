---
url: https://alchemy.run/gcp/frontend/vinext
title: "vinext"
description: "Deploy a vinext app (the Next.js API on Vite) to Cloud Run with GCP.Website.Vinext."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.Vinext` deploys a [vinext](https://vinext.dev/) app to Cloud Run. Alchemy runs vinext’s Vite build and serves it with vinext’s production server.

## Install

The build integration ships separately. It only runs at deploy time, so a dev dependency is enough:

```sh
pnpm add -D @alchemy.run/frontend-frameworks@latest
```

Docker has to be running when you deploy. The image is built on your machine.

## Deploy the site

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export const Website = GCP.Website.Vinext("Website");

export default Alchemy.Stack(
  "MySite",
  { providers: GCP.providers(), state: GCP.state() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

Run `alchemy deploy`. `site.url` is the Cloud Run service URL.

## Pass environment variables

```typescript
export const Website = GCP.Website.Vinext("Website");
export const Website = GCP.Website.Vinext("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
  },
});
```

Server code reads the value from `process.env.GREETING`.

ISR and `"use cache"` keep data in each instance’s memory, which other instances can’t see. Point `REDIS_URL` at a Redis server to share it:

```typescript
export const Website = GCP.Website.Vinext("Website", {
  env: { REDIS_URL: "redis://10.0.0.3:6379" },
});
```

[Memorystore](../data/memorystore.md) gives you a Redis server on a private IP.

## Tune Cloud Run

```typescript
export const Website = GCP.Website.Vinext("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#vinext) — every prop.
