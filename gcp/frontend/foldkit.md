---
url: https://alchemy.run/gcp/frontend/foldkit
title: "Foldkit"
description: "Deploy a Foldkit app to Cloud Run with GCP.Website.Foldkit — a Vite build with deep links that boot the app."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.Foldkit` deploys a [Foldkit](https://foldkit.dev/) app to Cloud Run. It is [`Vite`](vite.md) with single-page routing turned on, so a deep link loads `index.html` and boots the app.

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

export const Website = GCP.Website.Foldkit("Website");

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

## Project in a subdirectory

```typescript
export const Website = GCP.Website.Foldkit("Website", {
  rootDir: "applications/web",
});
```

## Serve a real 404 page

```typescript
export const Website = GCP.Website.Foldkit("Website", {
  assets: { notFoundHandling: "404-page" },
});
```

## Tune Cloud Run

```typescript
export const Website = GCP.Website.Foldkit("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#foldkit) — every prop.
