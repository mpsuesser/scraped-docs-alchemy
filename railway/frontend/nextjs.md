---
url: https://alchemy.run/railway/frontend/nextjs
title: "Next.js"
description: "Deploy a Next.js app to Railway with Railway.Website.Nextjs — next build plus a Node next({ dev: false }) container, and next dev under alchemy dev."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

`Railway.Website.Nextjs` deploys a [Next.js](https://nextjs.org/) app to Railway as a long-running Node process. It runs a real `next build`, then a serve entry that `import("next")` + `next({ dev: false }).prepare()` + `getRequestHandler()` on port 3000:

- one [`Railway.Project`](https://alchemy.run/railway/compute/projects) (created if `project` is omitted) and a [`Railway.Service`](https://alchemy.run/railway/compute/services) from a container image
- `.next` and `public/` baked into `/app`; `next`, `react`, and `react-dom` installed with `bun install`
- `url` is `https://{name}.up.railway.app`

This is **not** OpenNext — those wrappers target Lambda and workerd. ISR and `next/image` use Next’s own Node behavior.

App Router and Pages Router both work — server components, API routes, middleware, server actions, dynamic segments, streaming SSR, and `getServerSideProps` pages all run in the Node process.

## Install

The build integration is not bundled with alchemy. Install `@alchemy.run/frontend-frameworks`; the resource loads the package’s `/nextjs/node` export from your project at deploy time. It is only used at build time, so a dev dependency is enough. Your app still needs `next`, `react`, and `react-dom` as usual:

```sh
bun add -d @alchemy.run/frontend-frameworks
```

## Configure Next.js

Your `next.config.*` is loaded and honored as-is — the build runs a real `next build`, and Alchemy never rewrites the file. There is no adapter to install and no `open-next.config.ts` to write.

Pass `rootDir` when the Next.js project is not the stack root.

## Declare the Website

Declare the site as a module-level const (rather than inline in the Stack):

```typescript
import * as Railway from "alchemy/Railway";

export const Website = Railway.Website.Nextjs("Website");
```

## Add it to the Stack

Yield the site from your Stack and return its URL:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyNextjsSite",
  {
    providers: Railway.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`url` is the Service’s `https://{name}.up.railway.app` on deploy. `service` and `project` are set then and `undefined` under `alchemy dev`.

Pass an existing Project to share it with other Services:

```typescript
export const Website = Railway.Website.Nextjs("Website", {
  project: Site,
});
```

## Add environment variables

Process environment is the top-level `env` prop — plain strings, or `Redacted` secrets:

```typescript
export const Website = Railway.Website.Nextjs("Website", {
  env: {
    GREETING: "Hello from Alchemy!",
    API_BASE: "https://api.example.com",
  },
});
```

The values are copied onto `process.env` before `next build` / `next dev`, and onto the Service at deploy, so server code and `NEXT_PUBLIC_*` inlining see the same map in both modes.

## Read the environment in server code

The Next server is a plain Node process, so route handlers, server components, and server actions read the environment from `process.env`:

```typescript
export function GET() {
  return Response.json({ greeting: process.env.GREETING ?? "hello" });
}
```

## Local dev

```sh
bun alchemy dev
```

`alchemy dev` runs `next dev` (native HMR, Turbopack) instead of deploying; `site.url` is the local address and no Railway resources are created. Wrap the site in `Alchemy.remote()` to deploy the live Service even during dev.

## Custom domain

```typescript
const site = yield* Railway.Website.Nextjs("Web", {
  domain: "app.example.com",
});
```

Alchemy attaches a [`Railway.CustomDomain`](https://alchemy.run/railway/networking) on the Service. `url` becomes `https://app.example.com` instead of the generated `*.up.railway.app`. Railway issues a certificate once DNS is verified.

## Where next

- [`Railway.Website.Nextjs` reference](https://alchemy.run/providers/railway/website/nextjs) — every prop and attribute
- [Projects](https://alchemy.run/railway/compute/projects), [Services](https://alchemy.run/railway/compute/services), [Networking](https://alchemy.run/railway/networking)
