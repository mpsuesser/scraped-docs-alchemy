---
url: https://alchemy.run/gcp/frontend/waku
title: "Waku"
description: "Deploy a Waku (React Server Components) app to Cloud Run with GCP.Website.Waku."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.Waku` deploys a [Waku](https://waku.gg/) app to Cloud Run. The React Server Components server runs in the container. Static pages are served without their extension (`/about`).

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

export const Website = GCP.Website.Waku("Website");

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
export const Website = GCP.Website.Waku("Website");
export const Website = GCP.Website.Waku("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
  },
});
```

Server code reads the value from `process.env.GREETING`.

## Configure Waku

Your `waku.config.*` loads as-is. The `waku` prop overrides `srcDir`, `distDir`, or `basePath` for a deploy:

```typescript
export const Website = GCP.Website.Waku("Website", {
  waku: { basePath: "/app/" },
});
```

## Tune Cloud Run

```typescript
export const Website = GCP.Website.Waku("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#waku) — every prop.
