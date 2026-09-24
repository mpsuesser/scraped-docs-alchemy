---
url: https://alchemy.run/prisma/frontend/vinext
title: "Vinext"
description: "Deploy vinext to Prisma Compute with the shared Website API, native local development, and traced runtime dependencies."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

`Prisma.Website.Vinext` builds with the Vinext Vite API, then hosts vinext’s production Node server on **Bun in Prisma Compute**. Alchemy packages `dist/`, public assets, and locally installed runtime dependencies into a `tar.gz` artifact. There is no Dockerfile, registry, Wrangler configuration, or OpenNext build.

## Install

Keep `vinext`, `react`, `react-dom`, and `react-server-dom-webpack` in your application dependencies, and install `vite` and `@vitejs/plugin-rsc` for building. Add the Alchemy integration and dependency tracer:

```sh
bun add -d @alchemy.run/frontend-frameworks @vercel/nft
```

## Configure vinext

```typescript
import { defineConfig } from "vite";
import vinext from "vinext";

export default defineConfig({
  plugins: [vinext({ prerender: true })],
});
```

Alchemy injects the Node data-cache adapter during deployment. No Alchemy plugin is required in `vite.config.ts`. Configure deployment settings on `Prisma.Website.Vinext` and leave Cloudflare plugins out of the Vite config.

## Declare the Website

```typescript
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.Vinext("Website", {
  rootDir: ".",
});
```

A live deployment creates a database-less project when `project` is omitted. Pass an existing project to share it with other applications.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyVinextSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`site.url` is the Compute endpoint on deploy and the native dev-server URL locally. The result also exposes `compute`, `project`, and `domain`, matching other Prisma Website constructors.

## Add environment variables

```typescript
export const Website = Prisma.Website.Vinext("Website", {
  env: { GREETING: "Hello from vinext on Prisma!" },
});
```

`env` is available during build, development, and production. Server components, route handlers, and server actions read `process.env`. Public framework variables can be compiled into browser assets; keep secrets out of public keys.

## Configure the data cache

The default cache is process-local and does not survive a Compute restart or provide cross-instance consistency. For persistent ISR and `"use cache"`, provide an existing Redis endpoint:

```typescript
import * as Redacted from "effect/Redacted";

export const Website = Prisma.Website.Vinext("Website", {
  env: {
    REDIS_URL: Redacted.make("rediss://user:password@cache.example.com:6379"),
  },
});
```

The injected cache adapter uses Redis when `REDIS_URL` is set and memory otherwise. Prisma.Website does not provision Redis. The same Node adapter is used by Fly, Hetzner, and Railway.

## Local development

```sh
bun alchemy dev
```

The Website runs `vinext dev` with native HMR without creating a Prisma project, Compute service, or custom domain. Other resources declared in the Stack retain their own provider behavior. Apply `.pipe(Alchemy.remote())` to deploy the Website to Compute during development.

## Custom domains

```typescript
const site = yield* Prisma.Website.Vinext("Web", {
  domain: "app.example.com",
});
```

The app must use the project’s default branch. Configure the returned DNS records and verify domain status before routing traffic; see [Custom domains](websites.md#custom-domains).

## Packaging

Prisma runs the uploaded artifact without installing dependencies. `@vercel/nft` traces the installed application’s runtime graph. Native addons require a Linux-compatible build environment; a macOS deployment does not cross-compile them.

## Where next

- [vinext API reference](https://alchemy.run/providers/prisma/website#vinext).
- [Runnable example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-vinext).
- [Websites](websites.md) and [Compute apps](../compute/apps.md).
- [vinext on Cloudflare](../../cloudflare/frontend/vinext.md), [AWS](../../aws/frontend/vinext.md), [Fly](../../fly/frontend/vinext.md), [Hetzner](../../hetzner/frontend/vinext.md), and [Railway](../../railway/frontend/vinext.md).
