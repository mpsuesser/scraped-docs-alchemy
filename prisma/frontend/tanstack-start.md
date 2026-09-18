---
url: https://alchemy.run/prisma/frontend/tanstack-start
title: "TanStack Start"
description: "Deploy TanStack Start to Prisma Compute with Prisma.Website.TanStackStart — React or Solid SSR on Bun, with native Vite dev locally."
access_date: 2026-09-18T03:55:07.187Z
current_date: 2026-09-18T03:55:07.187Z
---

`Prisma.Website.TanStackStart` builds [TanStack Start](https://tanstack.com/start) for React or Solid through your project’s Vite pipeline. The shared Node target wraps its fetch handler as an HTTP server; SSR and client assets run on **Bun in Prisma Compute** from a `tar.gz` upload. There is no Docker image or registry.

## Install

Install the build-time integration; the resource loads `/tanstack-start` and `/tanstack-start/node` from your project:

```sh
bun add -d @alchemy.run/frontend-frameworks @vercel/nft
```

## Configure Vite

Keep the framework plugin and normal Vite plugins. For React:

```typescript
import tailwindcss from "@tailwindcss/vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [tailwindcss(), tanstackStart(), viteReact()],
});
```

The integration drives `builder.buildApp()` and makes the SSR bundle self-contained. No deployment adapter is required.

## Declare the Website

```typescript
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.TanStackStart("Website", {
  rootDir: "./app",
});
```

Omit `rootDir` for an app at `.`. Live deploy creates a database-less Prisma project if `project` is omitted; pass an existing project to share it with other apps.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyTanStackSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

Client assets are served first; misses go to SSR, server routes, or server functions. `site.url` is the Compute endpoint on deploy.

## Add environment variables

```typescript
export const Website = Prisma.Website.TanStackStart("Website", {
  rootDir: "./app",
  env: { API_BASE: "https://api.example.com" },
});
```

`env` is applied before build and dev and passed to Compute. Vite inlines `VITE_*` values into the browser bundle; keep secrets out of those keys. Sibling outputs can feed `env` inside the Stack generator.

## Read the environment in a server function

```typescript
import { createServerFn } from "@tanstack/react-start";

const getApiBase = createServerFn({ method: "GET" }).handler(() => ({
  apiBase: process.env.API_BASE ?? "unset",
}));
```

Server functions run on Bun and read `process.env`.

## Read the environment in a server route

```typescript
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/api/hello")({
  server: {
    handlers: {
      GET: () => new Response(process.env.API_BASE ?? "unset"),
    },
  },
});
```

Server routes use the same process environment.

## Build configuration

Your `vite.config.ts` is the build configuration. Keep Vite’s default output directory, `dist`; the integration reads `dist/server` and `dist/client`. Static files are served by Compute, not deployed as an independent CDN site.

## Local dev

`bun alchemy dev` starts TanStack Start’s native Vite server with HMR and creates no Prisma resources for the Website. Opt into live deployment during dev explicitly:

```typescript
export const Website = Prisma.Website.TanStackStart("Website").pipe(
  Alchemy.remote(),
);
```

## Custom domain

```typescript
const site = yield* Prisma.Website.TanStackStart("Web", {
  domain: "app.example.com",
});
```

`Prisma.CustomDomain` requires the app to be on the current default branch. Configure returned DNS records yourself and verify status before cutover; see [Custom domains](websites.md#custom-domains).

## Where next

- [TanStack Start API](https://alchemy.run/providers/prisma/website/tanstackstart).
- [TanStack Start example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-tanstack-start).
- [Websites](websites.md) and [Compute apps](../compute/apps.md).
