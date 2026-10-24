---
url: https://alchemy.run/gcp/frontend/tanstack-start
title: "TanStack Start"
description: "Deploy a TanStack Start app (React or Solid) to Cloud Run with GCP.Website.TanStackStart — your own vite build, served by a Node server."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.TanStackStart` deploys a [TanStack Start](https://tanstack.com/start) app, React or Solid, to Cloud Run. Alchemy runs your own `vite build` and serves the result from a Node server.

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

export const Website = GCP.Website.TanStackStart("Website");

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
export const Website = GCP.Website.TanStackStart("Website");
export const Website = GCP.Website.TanStackStart("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
  },
});
```

Server functions read it from `process.env`:

```typescript
const getGreeting = createServerFn({ method: "GET" }).handler(
  () => process.env.GREETING ?? "Hello!",
);
```

## Configure Vite

Your `vite.config.ts` needs no adapter. Keep the TanStack Start plugin as it is.

## Tune Cloud Run

```typescript
export const Website = GCP.Website.TanStackStart("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#tanstackstart) — every prop.
