---
url: https://alchemy.run/prisma/frontend/octane
title: "Octane"
description: "Deploy OctaneJS to Prisma Compute with Prisma.Website.Octane — the Node target on Bun, SSR plus client assets, and native Vite dev locally."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

`Prisma.Website.Octane` runs your [OctaneJS](https://octanejs.dev/) project’s Vite build. Octane builds the client and SSR bundles; the shared Node target wraps the fetch handler as an HTTP program. Both outputs are uploaded as `tar.gz` and run on **Bun in Prisma Compute**, not a Docker Node container.

## Install

Install the build-time integration; the resource loads `/octane` and `/octane/node` from your project:

```sh
bun add -d @alchemy.run/frontend-frameworks @vercel/nft
```

## Configure Octane

Keep native compiler and route settings in `octane.config.ts`, without an adapter:

```typescript
import { defineConfig, RenderRoute } from "@octanejs/vite-plugin";

export default defineConfig({
  router: {
    routes: [new RenderRoute({ path: "/", entry: ["App", "/src/App.tsx"] })],
  },
});
```

`Prisma.Website.Octane` selects hosting and automatically wraps Octane’s default native Node output as an HTTP server running on Bun in Compute. The legacy Node marker adapter remains optional for existing projects.

## Configure Vite

Put Vite plugins alongside Octane’s plugin in `vite.config.ts`:

```typescript
import { octane } from "@octanejs/vite-plugin";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [octane(), tailwindcss()],
});
```

## Declare the Website

```typescript
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.Octane("Website");
```

`rootDir` defaults to `"."`. Omit `project` for a database-less project created only on live deploy, or pass a project already in your Stack.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyOctaneSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`site.url` is the Compute endpoint on deploy and the local dev URL otherwise.

## Add environment variables

```typescript
export const Website = Prisma.Website.Octane("Website", {
  env: {
    GREETING: "Hello from Alchemy!",
    API_BASE: "https://api.example.com",
  },
});
```

Strings or `Redacted` values are applied before build and dev, and passed to Compute as process environment.

## Read the environment in server code

```typescript
new ServerRoute({
  path: "/api/greeting",
  methods: ["GET"],
  handler: async () => Response.json({
    greeting: process.env.GREETING ?? "hello",
  }),
});
```

Import `ServerRoute` from `@octanejs/vite-plugin` when adding this route. Middleware and route handlers run on Bun and read `process.env`.

## Asset routing

Files in `dist/client` are served first without invoking the Octane handler. Requests that miss the client assets go to Octane SSR. Both paths run in Compute; static assets are not a separate CDN deployment.

## Local dev

`bun alchemy dev` starts Octane’s Vite server with its in-process SSR, server routes, RPC, and HMR. The Website creates no Prisma resources. Use `.pipe(Alchemy.remote())` to opt into live deployment during dev.

## Custom domain

```typescript
const site = yield* Prisma.Website.Octane("Web", {
  domain: "app.example.com",
});
```

`Prisma.CustomDomain` requires the project’s current default branch. Configure its returned DNS records yourself and verify status before cutover; see [Custom domains](websites.md#custom-domains).

## Where next

- [Octane API](https://alchemy.run/providers/prisma/website#octane).
- [Octane example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-octane).
- [Websites](websites.md) and [Compute apps](../compute/apps.md).
