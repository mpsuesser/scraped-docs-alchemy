---
url: https://alchemy.run/aws/frontend/solidstart
title: "SolidStart"
description: "Deploy a SolidStart app to AWS with AWS.Website.SolidStart — SSR on a Lambda Web Function, S3 assets behind CloudFront, and SolidStart's own Vite dev server under alchemy dev."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`AWS.Website.SolidStart` deploys a [SolidStart](https://start.solidjs.com/) app to AWS. It runs your project’s own `vite build` and appends nitro’s `node` preset: the SSR server runs on a [Lambda Web Function](../compute/web-functions.md) and streams its responses, and client assets plus prerendered pages are served from a private S3 bucket through CloudFront. A CloudFront Function routes each request at the edge — uploaded files go to S3, everything else streams from the server.

## Install

The build integration is not bundled with alchemy. Install `@alchemy.run/frontend-frameworks`; the resource loads its `/solidstart` and `/solidstart/node` exports from your project at deploy time. It is only used at build time, so a dev dependency is enough:

```sh
pnpm add -D @alchemy.run/frontend-frameworks@latest
```

`@solidjs/vite-plugin-nitro-2` is the server half of a SolidStart build and must be a project dependency too:

```sh
pnpm add @solidjs/vite-plugin-nitro-2
```

## Configure Vite

Your `vite.config.ts` is just the SolidStart plugin — no adapter, no nitro wiring:

```typescript
import { solidStart } from "@solidjs/start/config";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [solidStart()],
});
```

Alchemy appends its own `nitroV2Plugin()` instance at build time, carrying nitro’s `node` preset (`aws-lambda`, streaming enabled, with `server: "function"`). Don’t register `nitroV2Plugin()` yourself — the appended instance would discard its options, so the build fails with an actionable error instead. Pass nitro options through the resource’s `nitro` prop; see [Prerendering](#prerendering) below.

## Declare the Website

Declare the site as a module-level const (rather than inline in the Stack):

```typescript
import * as AWS from "@alchemy.run/aws";

export const Website = AWS.Website.SolidStart("Website");
```

`rootDir` is the project root — the directory holding `package.json` and `vite.config.*`. It defaults to `"."`, so set it only when the app does not sit next to `alchemy.run.ts`:

```typescript
export const Website = AWS.Website.SolidStart("Website", {
  rootDir: "./app",
});
```

## Add it to the Stack

Yield the site from your Stack and return its URL:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MySolidStartSite",
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

## Add environment variables

The server function’s environment is configured under `env`:

```typescript
export const Website = AWS.Website.SolidStart("Website", {
  env: {
    API_BASE: "https://api.example.com",
  },
});
```

`env` values are set on the server on deploy and injected into the dev server’s process environment under `alchemy dev`, so server code reads the same values in both modes. Sibling props tune the server itself (`timeout`, `maxConcurrencyPerEnvironment`, `maxEnvironments`).

An output from another resource only exists once that resource is yielded, so declare the site inside the Stack generator instead:

```typescript
Effect.gen(function* () {
  const api = yield* Api;

  const site = yield* AWS.Website.SolidStart("Website", {
    env: { API_BASE: api.url },
  });

  return { url: site.url };
});
```

## Read the environment in server code

The SolidStart server is a plain Node server, so API routes, server functions, and SSR read the environment from `process.env`:

```typescript
export function GET() {
  return new Response(process.env.API_BASE ?? "unset");
}
```

Values prefixed `VITE_` are inlined into the client bundle at build time instead, as `import.meta.env.VITE_*` — set those in the shell that runs `alchemy deploy`, not in `env`.

## Prerendering

Nitro renders the routes you list at build time into `.output/public`. They are uploaded to S3 and served from the edge with no Lambda invocation:

```typescript
export const Website = AWS.Website.SolidStart("Website", {
  nitro: { prerender: { routes: ["/", "/about"] } },
});
```

`prerender` takes the routes to render (`routes`) and can crawl links from prerendered pages to prerender them too (`crawlLinks`). The `nitro` prop is the home for all nitro options (route rules, storage, …) — the integration owns the plugin instance, so they cannot live in `vite.config.ts`. Values must be JSON-serializable; `preset` is owned by the deploy target, and a foreign preset is a hard error.

## Run on a Lambda Function

Web Functions run in a subset of Regions. Elsewhere, set `server: "function"`: the app builds with the `/solidstart/aws` deploy target and runs on a Lambda Function behind a streaming Function URL, tuned with `memorySize`, `architecture`, and `runtime`:

```typescript
export const Website = AWS.Website.SolidStart("Website", {
  server: "function",
  memorySize: 2048,
});
```

## Local dev

```sh
bun alchemy dev
```

`alchemy dev` runs SolidStart’s own Vite dev server (native HMR) instead of deploying anything; `site.url` is the dev server’s `http://localhost:<port>` address and no AWS resources are created. Nitro plays no part in dev. Wrap the site in `Alchemy.remote()` to deploy the real infrastructure even during dev.

SolidStart’s plugin resolves its app root and route directory from the process working directory, so one `alchemy dev` process serves one SolidStart app at a time.

## Custom domain

```typescript
export const Website = AWS.Website.SolidStart("Website", {
  domain: { name: "app.example.com" },
});
```

The certificate is created in `us-east-1` (required by CloudFront), DNS validation records and the alias record are managed through Route 53. The hosted zone is inferred from the hostname — pass `hostedZoneId` to pin it when several zones could match.

CloudFront distributions take minutes to create. To serve several sites (or a site plus an API) from one distribution, attach the site to an [`AWS.Website.Router`](static-site.md#compose-sites-with-a-router):

The site takes the Router’s outputs, so both are yielded inside the Stack generator:

```typescript
Effect.gen(function* () {
  const router = yield* AWS.Website.Router("FrontDoor", {
    domain: { name: "example.com" },
  });

  const site = yield* AWS.Website.SolidStart("Website", {
    domain: { router },
  });

  return { url: site.url };
});
```
