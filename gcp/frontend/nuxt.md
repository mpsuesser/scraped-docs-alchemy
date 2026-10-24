---
url: https://alchemy.run/gcp/frontend/nuxt
title: "Nuxt"
description: "Deploy a Nuxt app to Cloud Run with GCP.Website.Nuxt — the nitro Node server in a container, prerendered pages served as static files."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.Nuxt` deploys a [Nuxt](https://nuxt.com/) app to Cloud Run. Nitro’s Node server runs in the container. Prerendered pages and static assets are served from the same image.

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

export const Website = GCP.Website.Nuxt("Website");

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
export const Website = GCP.Website.Nuxt("Website");
export const Website = GCP.Website.Nuxt("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
  },
});
```

Server routes read it from `process.env.GREETING`.

## Configure Nuxt

Your `nuxt.config.ts` loads as-is. The `nuxt` prop merges extra config over it for a single deploy:

```typescript
export const Website = GCP.Website.Nuxt("Website", {
  nuxt: { routeRules: { "/": { prerender: true } } },
});
```

Don’t set `nitro.preset`. Alchemy sets it.

## Tune Cloud Run

```typescript
export const Website = GCP.Website.Nuxt("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#nuxt) — every prop.
