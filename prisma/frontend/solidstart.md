---
url: https://alchemy.run/prisma/frontend/solidstart
title: "SolidStart"
description: "Deploy SolidStart to Prisma Compute with Prisma.Website.SolidStart — Nitro's Node target on Bun and SolidStart's native Vite dev server locally."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

`Prisma.Website.SolidStart` runs your [SolidStart](https://start.solidjs.com/) project’s Vite build with Nitro’s Node preset. SSR, client assets, and prerendered pages run on **Bun in Prisma Compute** from a `tar.gz` artifact, not a Docker image.

## Install

Install the build integration; the resource loads `/solidstart` and `/solidstart/node` from your project:

```sh
bun add -d @alchemy.run/frontend-frameworks @vercel/nft
```

Install the Nitro plugin dependency too:

```sh
bun add @solidjs/vite-plugin-nitro-2
```

## Configure Vite

```typescript
import { solidStart } from "@solidjs/start/config";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [solidStart(), tailwindcss()],
});
```

Alchemy appends `nitroV2Plugin()` with its Node preset at build time. Don’t register another Nitro plugin yourself: conflicting instances fail the build. Put Nitro overrides in the resource’s `nitro` bag.

## Declare the Website

```typescript
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.SolidStart("Website", {
  rootDir: "./app",
});
```

Omit `rootDir` for an app at `.`. Pass `project` to share an existing project; otherwise live deploy creates a database-less Prisma project.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MySolidStartSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`site.url` is the Compute endpoint on deploy and the local address in dev.

## Add environment variables

```typescript
export const Website = Prisma.Website.SolidStart("Website", {
  rootDir: "./app",
  env: { API_BASE: "https://api.example.com" },
});
```

Values are applied before build and dev and passed to Compute. Vite inlines `VITE_*` keys into the client bundle; keep secrets out of them. Sibling resource outputs can feed `env` inside the Stack generator.

## Read the environment in server code

```typescript
export function GET() {
  return new Response(process.env.API_BASE ?? "unset");
}
```

API routes, server functions, and SSR run on Bun and read `process.env`.

## Prerendering

```typescript
export const Website = Prisma.Website.SolidStart("Website", {
  nitro: { prerender: { routes: ["/", "/about"] } },
});
```

Nitro emits those pages into `.output/public`; Compute serves them as files without invoking SSR. `crawlLinks` can discover more routes. The `nitro` bag accepts serializable options such as route rules and storage, but the Node target owns `preset`.

## Local dev

`alchemy dev` runs SolidStart’s Vite server with HMR; Nitro is not part of the dev path and the Website creates no Prisma resources. SolidStart resolves the app root from the working directory, so run one SolidStart app per dev process.

Opt into the live deployment during dev explicitly:

```typescript
export const Website = Prisma.Website.SolidStart("Website").pipe(
  Alchemy.remote(),
);
```

## Custom domain

```typescript
const site = yield* Prisma.Website.SolidStart("Web", {
  domain: "app.example.com",
});
```

`Prisma.CustomDomain` requires the app to be on the current default branch. Configure returned DNS records yourself and verify status before cutover; see [Custom domains](websites.md#custom-domains).

## Where next

- [SolidStart API](https://alchemy.run/providers/prisma/website#solidstart).
- [SolidStart example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-solidstart).
- [Websites](websites.md) and [Compute apps](../compute/apps.md).
