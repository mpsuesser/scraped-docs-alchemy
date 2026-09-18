---
url: https://alchemy.run/prisma/frontend/nextjs
title: "Next.js"
description: "Deploy Next.js to Prisma Compute with Prisma.Website.Nextjs — a normal Next server on Bun, not OpenNext, and next dev locally."
access_date: 2026-09-18T03:55:07.187Z
current_date: 2026-09-18T03:55:07.187Z
---

`Prisma.Website.Nextjs` runs a real [Next.js](https://nextjs.org/) `next build`, then hosts a normal Next server on **Bun in Prisma Compute**. The shared Node target uses `next({ dev: false })`, `prepare()`, and `getRequestHandler()`. The build output, public assets, and runtime dependencies are packaged for a `tar.gz` upload; there is no Docker image or registry.

This is **not OpenNext**. App Router and Pages Router use Next’s own server, including server components, API routes, server actions, and SSR. ISR and image optimization follow the normal Next server path, not an external cache or image service provisioned by Alchemy.

## Install

Install the build-time integration; the resource loads `/nextjs/node` from your project. Keep `next`, `react`, and `react-dom` as app dependencies.

```sh
bun add -d @alchemy.run/frontend-frameworks @vercel/nft
```

## Configure Next.js

`next.config.*` loads natively. There is no adapter to install and no `open-next.config.ts`. Pass `rootDir` if your Next app is not at the Stack root.

## Declare the Website

```typescript
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.Nextjs("Website");
```

A live deployment creates a database-less Prisma project if you omit `project`. Pass an existing project to share it with other apps.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyNextjsSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`site.url` is the Compute endpoint on deploy and Next’s local URL in dev.

## Add environment variables

```typescript
export const Website = Prisma.Website.Nextjs("Website", {
  env: {
    GREETING: "Hello from Alchemy!",
    API_BASE: "https://api.example.com",
  },
});
```

Values are applied before `next build` and `next dev`, and passed to Compute. `NEXT_PUBLIC_*` values are inlined into the browser bundle; other keys remain server-side. `env` accepts strings or `Redacted` secrets.

## Read the environment in server code

```typescript
export function GET() {
  return Response.json({ greeting: process.env.GREETING ?? "hello" });
}
```

Route handlers, server components, and server actions read the Bun process environment with `process.env`.

## Local dev

```sh
bun alchemy dev
```

Alchemy runs `next dev` with native HMR rather than deploying. The Website creates no Prisma resources. Use `.pipe(Alchemy.remote())` to deploy live during dev instead.

## Custom domain

```typescript
const site = yield* Prisma.Website.Nextjs("Web", {
  domain: "app.example.com",
});
```

This creates `Prisma.CustomDomain` and changes `site.url` to the HTTPS hostname. The app must be on the project’s current default branch. Configure returned DNS records yourself and verify domain status before cutover; see [Custom domains](websites.md#custom-domains).

## Where next

- [Next.js API](https://alchemy.run/providers/prisma/website/nextjs).
- [Next.js example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-nextjs).
- [Websites](websites.md) and [Compute apps](../compute/apps.md).
- [Connections](../data/connections.md) — database environment wiring.
