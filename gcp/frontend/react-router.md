---
url: https://alchemy.run/gcp/frontend/react-router
title: "React Router"
description: "Deploy a React Router v7 app in framework mode to Cloud Run with GCP.Website.ReactRouter."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.ReactRouter` deploys a [React Router](https://reactrouter.com/) v7 app in framework mode to Cloud Run. Alchemy runs your own `vite build` and serves the app from a Node server.

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

export const Website = GCP.Website.ReactRouter("Website");

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
export const Website = GCP.Website.ReactRouter("Website");
export const Website = GCP.Website.ReactRouter("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
  },
});
```

Loaders read it from `process.env`:

```typescript
export function loader() {
  return { greeting: process.env.GREETING ?? "Hello!" };
}
```

## Configure Vite

Your `vite.config.ts` needs no adapter. React Server Components are not supported yet: the build fails with a clear error if it emits more than one server entry.

## Tune Cloud Run

```typescript
export const Website = GCP.Website.ReactRouter("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#reactrouter) — every prop.
