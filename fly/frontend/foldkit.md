---
url: https://alchemy.run/fly/frontend/foldkit
title: "Foldkit"
description: "Deploy a Foldkit app to Fly with Fly.Website.Foldkit — a client-only Vite SPA on a Node static-file Service, fly.dev over shared IPv4, and Vite's own dev server under alchemy dev."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

[Foldkit](https://foldkit.dev/) is an Elm-architecture frontend framework built on Effect. Its apps are client-only Vite projects — the Foldkit Vite plugin only adds HMR and devtools wiring — so `Fly.Website.Foldkit` is [`Fly.Website.Vite`](vite.md) with SPA fallback to `index.html`. Deep links boot the app and the Foldkit router takes over.

Deploy creates a [`Fly.App`](https://alchemy.run/fly/compute/apps) (unless you pass `app`), a [`Fly.Service`](https://alchemy.run/fly/compute/services) running a static-file Node server on port 3000, and a shared IPv4 so `https://{app}.fly.dev` answers.

## Install

The build integration is not bundled with alchemy. Install `@alchemy.run/frontend-frameworks`; the resource loads its `/vite` and `/vite/node` exports from your project at deploy time. It is only used at build time, so a dev dependency is enough:

```sh
bun add -d @alchemy.run/frontend-frameworks
```

## Configure Vite

Your Vite config stays what Foldkit’s setup gives you. Vite plugins like Tailwind go in `plugins` as usual:

```typescript
import { foldkit } from "@foldkit/vite-plugin";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [foldkit(), tailwindcss()],
  optimizeDeps: {
    entries: ["src/entry.ts"],
  },
});
```

Alchemy runs Vite programmatically on the project root. The Foldkit plugin and the rest of your setup are preserved as-is.

## Declare the Website

Declare the site as a module-level const (rather than inline in the Stack):

```typescript
import * as Fly from "alchemy/Fly";

export const Website = Fly.Website.Foldkit("Website");
```

For an app in a subdirectory of a monorepo, point `rootDir` at it:

```typescript
export const Website = Fly.Website.Foldkit("Website", {
  rootDir: "applications/web",
});
```

## Add it to the Stack

Yield the site from your Stack and return its URL:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyFoldkitSite",
  {
    providers: Fly.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`site.url` is `https://{app}.fly.dev`. The site also exposes `app`, `service`, `ip`, and `certificate` — they are `undefined` under `alchemy dev`.

## Add environment variables

A Foldkit SPA has no server of its own, so anything it needs from the rest of your Stack is baked into the bundle at build time — pass a `VITE_` -prefixed key in top-level `env`. Values must be strings (or `Redacted`); resolve other resources’ URLs before passing them.

```typescript
export const Website = Fly.Website.Foldkit("Website", {
  env: {
    VITE_API_URL: "https://api.example.com",
  },
});
```

The values are copied onto `process.env` before the Vite build and onto the Machine. Client code reads the inlined value:

```typescript
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
}
```

```typescript
const apiUrl = import.meta.env.VITE_API_URL;
```

## Deep links

A Foldkit app that uses URL routing (`Runtime.makeApplication` with `route`, `onUrlRequest`, and `onUrlChange`) resolves routes on the client, so a deep link like `/counter/42` arrives as a request for a file that doesn’t exist.

`Foldkit` defaults `assets.notFoundHandling` to `"single-page-application"`, which returns `index.html` (200) for unmatched paths so the Foldkit runtime can resolve the route. Use `"404-page"` to serve a real 404 instead:

```typescript
export const Website = Fly.Website.Foldkit("Website", {
  assets: { notFoundHandling: "404-page" },
});
```

## Local dev

`alchemy dev` runs Vite’s own dev server (native HMR, Foldkit devtools) instead of deploying; `site.url` is the local address and no Fly App, Service, IP, or certificate is created. Wrap the site in `Alchemy.remote()` to deploy the live Service even during dev.

Pin the dev server’s address when several apps run in one Stack:

```typescript
export const Website = Fly.Website.Foldkit("Website", {
  dev: { host: "127.0.0.1", port: 5180, strictPort: true },
});
```

## Custom domain

Pass a hostname to request ACME (`Fly.Certificate`) on the App. `url` becomes `https://{domain}`. v1 expects that DNS already points at the App.

```typescript
const site = yield* Fly.Website.Foldkit("Web", {
  domain: "app.example.com",
});
```

## Where next

- [`Foldkit` reference](https://alchemy.run/providers/fly/website/foldkit) — every prop and attribute
- [Vite](vite.md) — the same static-file Service for any Vite app
- [Websites](websites.md) — the rest of the Fly frontend family
- [Apps](https://alchemy.run/fly/compute/apps), [Services](https://alchemy.run/fly/compute/services), [IPs & certificates](https://alchemy.run/fly/networking)
