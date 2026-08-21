---
url: https://alchemy.run/aws/frontend/nuxt
title: "Nuxt"
description: "Deploy a Nuxt app to AWS with AWS.Website.Nuxt — nitro's aws-lambda preset on a streaming Lambda Function URL, S3 assets behind CloudFront, and Nuxt's own dev server under alchemy dev."
access_date: 2026-08-21T19:05:43.655Z
current_date: 2026-08-21T19:05:43.655Z
---

`AWS.Website.Nuxt` deploys a [Nuxt](https://nuxt.com/) app to AWS. It builds the app through your project’s own `@nuxt/kit` with nitro’s `aws-lambda` preset: the nitro server runs on a Lambda Function URL with response streaming, and client assets plus prerendered pages are served from a private S3 bucket through CloudFront. A CloudFront Function routes each request at the edge — uploaded files go to S3, everything else streams from the Lambda. There is no `nitro.preset` to edit and no build command to run.

## Install

The build integration is not bundled with alchemy. Install `@alchemy.run/frontend-frameworks`; the resource loads its `/nuxt` and `/nuxt/aws` exports from your project at deploy time. It is only used at build time, so a dev dependency is enough:

```sh
bun add -d @alchemy.run/frontend-frameworks
```

## Configure Nuxt

Your `nuxt.config.ts` loads natively — modules, layers, and all — so configure Nuxt exactly as you would outside Alchemy:

```typescript
export default defineNuxtConfig({
  modules: ["@nuxtjs/tailwindcss"],
  routeRules: {
    "/about": { prerender: true },
  },
});
```

Deploy-specific overrides are merged over it via the `nuxt` prop (the override wins) — see [Prerendering](#prerendering) below for an example.

Don’t set `nitro.preset` — the AWS deploy target owns the preset (`aws-lambda` with streaming enabled), and a foreign preset is a hard error.

## Declare the Website

Declare the site as a module-level const (rather than inline in the Stack):

```typescript
import * as AWS from "alchemy/AWS";

export const Website = AWS.Website.Nuxt("Website");
```

## Add it to the Stack

Yield the site from your Stack and return its URL:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyNuxtSite",
  {
    providers: AWS.providers(),
    state: AWS.state(),
  },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

See [examples/aws-website-nuxt](https://github.com/alchemy-run/alchemy/tree/main/examples/aws-website-nuxt) for the checked-in example.

## Add environment variables

The server function’s environment is configured under `server.environment` — plain values, or outputs from other resources in the Stack:

```typescript
export const Website = AWS.Website.Nuxt("Website", {
  server: {
    memorySize: 2048,
    environment: {
      NUXT_PUBLIC_API_BASE: api.url,
    },
  },
});
```

`environment` values are set on the Lambda on deploy and injected into the dev server’s process environment under `alchemy dev`, so server code reads the same values in both modes. The other `server` fields tune the Lambda itself (`memorySize`, `timeout`, `architecture`, `runtime`).

## Read the environment in server code

On AWS the nitro server runs in a plain Node Lambda, so server routes and SSR read the environment from `process.env` — or, the more idiomatic Nuxt way, through `runtimeConfig` overridden by `NUXT_` -prefixed environment variables:

```typescript
export default defineEventHandler(() => {
  return { greeting: process.env.GREETING ?? "hello" };
});
```

## Prerendering

Routes marked for prerendering in `routeRules` (or via `nitro.prerender`) render at build time into `.output/public` and are uploaded to S3, served from the edge with no Lambda invocation:

```typescript
export const Website = AWS.Website.Nuxt("Website", {
  nuxt: {
    routeRules: {
      "/about": { prerender: true },
    },
  },
});
```

Nitro’s `isr` route rule is implemented only by the Vercel and Netlify presets. On AWS Lambda it is silently ignored at build time, and the route renders on demand in the Lambda like any other SSR route. Use `prerender` for build-time static routes, or `cache` route rules for runtime caching.

## Local dev

```sh
bun alchemy dev
```

`alchemy dev` runs Nuxt’s own dev server (nitro dev, full HMR) instead of deploying anything; `site.url` is the dev server’s `http://localhost:<port>` address and no AWS resources are created. Wrap the site in `Alchemy.remote()` to deploy the real infrastructure even during dev.

## Custom domain

```typescript
const site = yield* AWS.Website.Nuxt("Web", {
  domain: {
    name: "app.example.com",
    hostedZoneId: zone.hostedZoneId,
  },
});
```

The certificate is created in `us-east-1` (required by CloudFront), DNS validation records and the alias record are managed through Route 53.

CloudFront distributions take minutes to create. To serve several sites (or a site plus an API) from one distribution, attach the site to an [`AWS.Website.Router`](static-site.md#compose-sites-with-a-router):

```typescript
const router = yield* AWS.Website.Router("FrontDoor", {
  domain: { name: "example.com", hostedZoneId: zone.hostedZoneId },
});

const site = yield* AWS.Website.Nuxt("Web", {
  domain: { router },
});
```
