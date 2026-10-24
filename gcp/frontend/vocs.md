---
url: https://alchemy.run/gcp/frontend/vocs
title: "Vocs"
description: "Deploy a Vocs documentation site to Cloud Run with GCP.Website.Vocs."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.Vocs` deploys a [Vocs](https://vocs.dev/) documentation site to Cloud Run. Static pages are served first, and everything else is rendered by the server.

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

export const Docs = GCP.Website.Vocs("Docs");

export default Alchemy.Stack(
  "MySite",
  { providers: GCP.providers(), state: GCP.state() },
  Effect.gen(function* () {
    const site = yield* Docs;
    return { url: site.url };
  }),
);
```

Run `alchemy deploy`. `site.url` is the Cloud Run service URL.

## Custom output directory

If `vocs.config.*` changes `outDir`, pass the same value:

```typescript
export const Docs = GCP.Website.Vocs("Docs", {
  rootDir: "./docs",
  outDir: "build",
});
```

## Tune Cloud Run

```typescript
export const Docs = GCP.Website.Vocs("Docs", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#vocs) — every prop.
