---
url: https://alchemy.run/gcp/frontend/nextjs
title: "Next.js"
description: "Deploy a Next.js app to Cloud Run with GCP.Website.Nextjs — Next's own production server in a container, no OpenNext."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website.Nextjs` deploys a [Next.js](https://nextjs.org/) app to Cloud Run. Alchemy runs `next build` and serves the app with Next’s own production server. There is no OpenNext: App Router, route handlers, and server components run exactly as they do on `next start`.

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

export const Website = GCP.Website.Nextjs("Website");

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
export const Website = GCP.Website.Nextjs("Website");
export const Website = GCP.Website.Nextjs("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
  },
});
```

Dynamic pages and route handlers read it from `process.env`:

```typescript
export function GET() {
  return Response.json({ greeting: process.env.GREETING });
}
```

## Configure Next.js

Your `next.config.*` is used as-is. Alchemy ships `.next/`, `public/`, and the config into the image and installs `next`, `react`, and `react-dom` there.

## Caching

Each Cloud Run instance keeps its own Next.js cache in memory. Instances don’t share it, so revalidated pages can differ between instances for a moment. Keep one warm instance, or cap `maxInstanceCount`, if that matters.

## Tune Cloud Run

```typescript
export const Website = GCP.Website.Nextjs("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
});
```

One warm instance means no cold start for the first visitor. See [Websites](websites.md#tune-cloud-run) for the other settings.

## Local development

`alchemy dev` runs the framework’s dev server with hot reload and creates nothing in the cloud. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Example

[`examples/gcp-website-nextjs`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-website-nextjs) has a dynamic page, a route handler, Tailwind, and a live test.

## Where next

- [Websites](websites.md) — the settings every site shares.
- [Website reference](https://alchemy.run/providers/gcp/website#nextjs) — every prop.
