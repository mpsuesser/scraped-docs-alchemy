---
url: https://alchemy.run/cloudflare/frontend/vinext
title: "Vinext"
description: "Deploy vinext to Cloudflare Workers with the shared Website API, native local development, and KV-backed caching."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

`Cloudflare.Website.Vinext` builds a [vinext](https://vinext.dev/) App Router application with Vite and deploys its React Server Components Worker and client assets. Alchemy injects its Cloudflare Vite plugin, prerenders routes, and seeds the managed KV cache. There is no Wrangler configuration or OpenNext build.

## Install

Keep `vinext`, `react`, `react-dom`, and `react-server-dom-webpack` in your application dependencies, and install `vite` and `@vitejs/plugin-rsc` for building. Add the Alchemy integration:

```sh
bun add -d @alchemy.run/frontend-frameworks
```

## Configure vinext

```typescript
import { defineConfig } from "vite";
import vinext from "vinext";

export default defineConfig({
  plugins: [vinext({ prerender: true })],
});
```

Alchemy injects its Cloudflare runtime and KV cache plugins. No Alchemy plugin is required in `vite.config.ts`; leave `@cloudflare/vite-plugin` out as well. Deployment settings belong on `Cloudflare.Website.Vinext`.

## Declare the Website

```typescript
import * as Cloudflare from "alchemy/Cloudflare";

export const Website = Cloudflare.Website.Vinext("Website", {
  rootDir: ".",
});

export type WebsiteEnv = Cloudflare.InferEnv<typeof Website>;
```

## Add the Worker entry

The default entry is `worker/index.ts`. Set `main` to use a different path.

```typescript
import handler from "vinext/server/fetch-handler";
import type { WebsiteEnv } from "../alchemy.run";

export default {
  fetch(request: Request, env: WebsiteEnv, ctx: ExecutionContext) {
    return handler.fetch(request, env, ctx);
  },
};
```

This Worker entry is Cloudflare-specific. The other platforms generate a production server or Lambda entry instead.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyVinextSite",
  { providers: Cloudflare.providers(), state: Cloudflare.state() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

## Add environment variables

```typescript
export const Website = Cloudflare.Website.Vinext("Website", {
  env: { GREETING: "Hello from vinext on Cloudflare!" },
});
```

Server components, route handlers, and server actions read bindings through `import { env } from "cloudflare:workers"`. Public framework variables can be compiled into browser assets; keep secrets out of public keys.

## Configure the data cache

Alchemy provisions `VINEXT_KV_CACHE`, enables Workers Cache, and binds `CF_VERSION_METADATA`. Keep these managed bindings out of `env`.

The resource automatically injects the KV adapter for ISR and `"use cache"`. Deployment uploads prerendered HTML and React Server Components payloads into KV. During prerendering, where the binding is absent, the adapter falls back to memory. Redis and S3 adapters are for other platforms.

## Local development

```sh
bun alchemy dev
```

The Website runs the native Vite development server with HMR and local Worker bindings. Other resources declared in the Stack retain their own provider behavior. Apply `.pipe(Alchemy.remote())` to deploy the Website during development.

## Custom domains

```typescript
const site = yield* Cloudflare.Website.Vinext("Web", {
  domain: "app.example.com",
});
```

Use a zone managed by your Cloudflare account; see [Frontends](frontends.md) for shared Website options.

## Where next

- [vinext API reference](https://alchemy.run/providers/cloudflare/website#vinext).
- [Runnable example](https://github.com/alchemy-run/alchemy/tree/main/examples/cloudflare-website-vinext), including server actions, D1, and KV.
- [vinext on AWS](../../aws/frontend/vinext.md), [Fly](../../fly/frontend/vinext.md), [Hetzner](../../hetzner/frontend/vinext.md), [Railway](../../railway/frontend/vinext.md), and [Prisma](../../prisma/frontend/vinext.md).
