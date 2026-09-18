---
url: https://alchemy.run/prisma/frontend/foldkit
title: "Foldkit"
description: "Deploy Foldkit to Prisma Compute with Prisma.Website.Foldkit — a Vite SPA served on Bun, deep-link fallback, and native HMR locally."
access_date: 2026-09-18T03:55:07.187Z
current_date: 2026-09-18T03:55:07.187Z
---

[Foldkit](https://foldkit.dev/) is an Elm-architecture frontend framework built on Effect. Its apps are client-only Vite projects, so `Prisma.Website.Foldkit` is [Vite](vite.md) with SPA fallback to `index.html`. Deep links boot the app and its router takes over.

The client build and generated server are uploaded as `tar.gz` and served on **Bun in Prisma Compute**. There is no Foldkit SSR handler, but the deployment still runs a Compute static-file server; no Docker image or registry is needed.

## Install

Install the build-time integration; the resource loads `/vite` and `/vite/node` from your frontend project:

```sh
bun add -d @alchemy.run/frontend-frameworks @vercel/nft
```

## Configure Vite

Keep Foldkit’s Vite setup and your existing plugins:

```typescript
import { foldkit } from "@foldkit/vite-plugin";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [foldkit(), tailwindcss()],
  optimizeDeps: { entries: ["src/entry.ts"] },
});
```

## Declare the Website

```typescript
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.Foldkit("Website", {
  rootDir: "applications/web",
});
```

Omit `rootDir` for an app at `.`. Omit `project` to create a database-less Prisma project on live deploy, or pass an existing project.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyFoldkitSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`site.url` is the Compute endpoint on deploy and Vite’s local URL in dev.

## Add environment variables

```typescript
export const Website = Prisma.Website.Foldkit("Website", {
  rootDir: "applications/web",
  env: {
    VITE_API_URL: "https://api.example.com",
  },
});
```

`env` is applied before build and dev, and passed to Compute. The SPA only sees values Vite inlines at build time. `VITE_*` keys are public, so never put secrets in them.

## Read the environment

```typescript
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
}
```

This declares the client-visible key for TypeScript.

```typescript
const apiUrl = import.meta.env.VITE_API_URL;
```

Vite replaces the lookup with the build-time value.

## Deep links

`assets.notFoundHandling` defaults to `"single-page-application"`. A path like `/counter/42` returns `index.html` with status 200, letting Foldkit’s client router resolve it. For real 404 responses instead:

```typescript
export const Website = Prisma.Website.Foldkit("Website", {
  assets: { notFoundHandling: "404-page" },
});
```

## Local dev

`bun alchemy dev` runs Vite with Foldkit HMR and devtools. The Website creates no Prisma resources. `.pipe(Alchemy.remote())` selects the live path during dev. Pin the local address when running several apps:

```typescript
export const Website = Prisma.Website.Foldkit("Website", {
  dev: { host: "127.0.0.1", port: 5180, strictPort: true },
});
```

## Custom domain

```typescript
const site = yield* Prisma.Website.Foldkit("Web", {
  domain: "app.example.com",
});
```

This creates `Prisma.CustomDomain`; the app must be on the project’s current default branch. Configure the returned DNS records yourself and verify status before cutover. See [Custom domains](websites.md#custom-domains).

## Where next

- [Foldkit API](https://alchemy.run/providers/prisma/website/foldkit).
- [Foldkit example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-foldkit).
- [Vite](vite.md), [Websites](websites.md), and [Compute apps](../compute/apps.md).
