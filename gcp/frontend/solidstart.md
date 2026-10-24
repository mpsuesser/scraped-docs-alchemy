---
url: https://alchemy.run/gcp/frontend/solidstart
title: "SolidStart"
description: "Deploy a SolidStart app to Cloud Run with GCP.Website.SolidStart — SSR in a container, prerendered pages served as static files."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.SolidStart` deploys a [SolidStart](https://start.solidjs.com/) app to Cloud Run. The server renders pages in the container, and prerendered pages are served as static files.

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

export const Website = GCP.Website.SolidStart("Website");

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
export const Website = GCP.Website.SolidStart("Website");
export const Website = GCP.Website.SolidStart("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
  },
});
```

Server code reads the value from `process.env.GREETING`.

## Prerender routes

Alchemy adds the nitro plugin to your build, so nitro options go on the `nitro` prop. Remove `nitroV2Plugin()` from `vite.config.*`:

```typescript
export const Website = GCP.Website.SolidStart("Website", {
  nitro: { prerender: { routes: ["/", "/about"] } },
});
```

## Tune Cloud Run

```typescript
export const Website = GCP.Website.SolidStart("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#solidstart) — every prop.
