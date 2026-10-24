---
url: https://alchemy.run/gcp/frontend/websites
title: "Websites"
description: "Deploy Vite, Astro, Next.js, Nuxt, SvelteKit, TanStack Start, React Router, SolidStart, Waku, vinext, Octane, Vocs, or any static build to Cloud Run with GCP.Website."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Website` deploys a frontend to Cloud Run. Each resource runs your framework’s build, bakes the server and the static files into one Node image, and serves it from a public Cloud Run service. There is no Dockerfile and no adapter to install. Your framework’s own config file loads as-is.

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MySite",
  { providers: GCP.providers(), state: GCP.state() },
  Effect.gen(function* () {
    const site = yield* GCP.Website.Astro("Website");
    return { url: site.url };
  }),
);
```

`site.url` is the service’s `https://….run.app` URL.

## Install

The build integration ships separately. It only runs at deploy time, so a dev dependency is enough:

```sh
pnpm add -D @alchemy.run/frontend-frameworks@latest
```

Docker has to be running when you deploy. The image is built on your machine.

## Pick your framework

| Framework | Resource | Guide |
| --- | --- | --- |
| React / Vite SPA | `Vite` | [React SPA](vite-spa.md) |
| Vue | `Vite` | [Vue](vue.md) |
| Any Vite app | `Vite` | [Vite](vite.md) |
| Foldkit | `Foldkit` | [Foldkit](foldkit.md) |
| Astro | `Astro` | [Astro](astro.md) |
| Next.js | `Nextjs` | [Next.js](nextjs.md) |
| vinext | `Vinext` | [vinext](vinext.md) |
| Nuxt | `Nuxt` | [Nuxt](nuxt.md) |
| SvelteKit | `SvelteKit` | [SvelteKit](sveltekit.md) |
| TanStack Start | `TanStackStart` | [TanStack Start](tanstack-start.md) |
| React Router | `ReactRouter` | [React Router](react-router.md) |
| SolidStart | `SolidStart` | [SolidStart](solidstart.md) |
| Waku | `Waku` | [Waku](waku.md) |
| Octane | `Octane` | [Octane](octane.md) |
| Vocs | `Vocs` | [Vocs](vocs.md) |
| Hugo, Zola, any static generator | `StaticSite` | [Static sites](static-site.md) |

Use the resource named after your framework. Use `Vite` for a client-only app. Use `StaticSite` for any build command that writes a directory of files.

## Pass environment variables

`env` sets the server’s environment. Values can be Outputs from other resources:

```typescript
const site = yield* GCP.Website.Nuxt("Website", {
  env: {
    GREETING: "Hello from Cloud Run!",
    API_URL: api.uri,
  },
});
```

Server code reads them from `process.env`.

## Tune Cloud Run

Every site takes the same Cloud Run settings:

```typescript
const site = yield* GCP.Website.Nextjs("Website", {
  scaling: { minInstanceCount: 1 },
  resources: { limits: { cpu: "1", memory: "1Gi" } },
  timeout: "60s",
});
```

`minInstanceCount: 1` keeps one instance warm, so the first visitor never waits for a cold start.

## Make it private

Sites are public by default. Set `public: false` to require a Google identity token, for an internal tool behind [IAP](https://cloud.google.com/iap) or a load balancer:

```typescript
const site = yield* GCP.Website.Vite("Admin", { public: false });
```

## Skip unchanged builds

Alchemy hashes your project’s files and skips the build and the image push when nothing changed. Limit the hash to the files that matter:

```typescript
const site = yield* GCP.Website.Astro("Website", {
  memo: {
    include: ["src/**", "public/**", "package.json", "astro.config.ts"],
  },
});
```

## Local development

`alchemy dev` runs your framework’s own dev server with hot reload and creates nothing in the cloud. `site.url` is the local address. Wrap the site in `Alchemy.remote()` to deploy it for real during dev.

## Where next

- [Astro](astro.md) and [Next.js](nextjs.md) — both have runnable examples.
- [Static sites](static-site.md) — a build command, or a public Cloud Storage bucket.
- [Website reference](https://alchemy.run/providers/gcp/website) — every prop.
