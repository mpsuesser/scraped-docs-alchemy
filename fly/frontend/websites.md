---
url: https://alchemy.run/fly/frontend/websites
title: "Websites"
description: "Deploy Vite, Astro, Next.js, Nuxt, React Router, SolidStart, SvelteKit, TanStack Start, Waku, Octane, Foldkit, Vocs, or any static build to Fly with first-class Website resources."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

Alchemy deploys frontends to Fly with a family of `Fly.Website`
resources. Each one builds your project programmatically and
deploys it as a Node [Service](https://alchemy.run/fly/compute/services) on a Fly
[App](https://alchemy.run/fly/compute/apps) — a Machine listening on port 3000, plus
a shared IPv4 so `https://{app}.fly.dev` answers — with no
`fly.toml` and no adapter to put in the framework config. Your
framework's own config file (`vite.config.ts`, `astro.config.*`,
`nuxt.config.ts`, ...) loads natively; Alchemy layers its Node
container integration on top.

- [`Vite`](vite.md) — client-only Vite apps (React,
  Vue, Solid SPAs, `index.html` multi-page sites); a tiny
  static-file server serves the `vite build` output.
- [`Astro`](astro.md) — Astro sites, server-rendered
  on Node or fully static; your `astro.config.*` loads natively.
- [`Nextjs`](nextjs.md) — Next.js as a long-running
  Node process (`next build`, then `next({ dev: false })`). Not
  OpenNext.
- [`Nuxt`](nuxt.md) — Nuxt apps through nitro's Node
  server; your `nuxt.config.ts` loads natively. Do not set
  `nitro.preset`.
- [`ReactRouter`](react-router.md) — React Router v7 in
  framework mode, built through your own `vite build`.
- [`SolidStart`](solidstart.md) — SolidStart 2.x SSR;
  Alchemy appends nitro's `node` preset to your `vite build`.
- [`SvelteKit`](sveltekit.md) — SvelteKit SSR plus
  prerendered assets; the Node adapter is injected. Do not set
  `kit.adapter`.
- [`TanStackStart`](tanstack-start.md) — TanStack Start
  (React or Solid), built through your own `vite build`.
- [`Waku`](waku.md) — Waku (React Server Components)
  on Node, with SSG pages served extensionless.
- [`Octane`](octane.md) — OctaneJS SSR. Select
  `node()` from `@alchemy.run/frontend-frameworks/octane/node-adapter`
  in `octane.config.ts`.
- [`Foldkit`](foldkit.md) — client-only Foldkit SPA;
  deep links fall back to `index.html`.
- [`Vocs`](vocs.md) — prerendered Vocs docs; extensionless
  pages (`/about` → `about/index.html`).
- [`StaticSite`](static-site.md) — any build command's
  output directory (Hugo, Zola, Eleventy).

Each resource creates a `Fly.App` when you omit `app`, a
`Fly.Service` on port 3000, and a `shared_v4`
[`IpAssignment`](https://alchemy.run/fly/networking) so `{app}.fly.dev` answers.
`domain` requests ACME via [`Certificate`](https://alchemy.run/fly/networking#certificates)
(existing DNS only for v1); `url` becomes `https://{domain}`.
The return is `{ url, app, service, ip, certificate }` —
everything except `url` is `undefined` under `alchemy dev`.

Shared props: `rootDir`, `memo`, `env` (Machine process
environment — not Worker bindings), `dev`, `domain`, `app`.
Builds are memoized by content-hashing the input files — an
unchanged project skips the build.

## What's supported

| Framework | Resource | Guide |
| --- | --- | --- |
| React / Vue / Solid SPA | `Vite` | [Vite](vite.md) |
| TanStack Start (React & Solid) | `TanStackStart` | [TanStack Start](tanstack-start.md) |
| React Router | `ReactRouter` | [React Router](react-router.md) |
| SolidStart | `SolidStart` | [SolidStart](solidstart.md) |
| Astro | `Astro` | [Astro](astro.md) |
| Next.js | `Nextjs` | [Next.js](nextjs.md) |
| Nuxt | `Nuxt` | [Nuxt](nuxt.md) |
| SvelteKit | `SvelteKit` | [SvelteKit](sveltekit.md) |
| Waku | `Waku` | [Waku](waku.md) |
| OctaneJS | `Octane` | [Octane](octane.md) |
| Foldkit | `Foldkit` | [Foldkit](foldkit.md) |
| Vocs | `Vocs` | [Vocs](vocs.md) |
| Zola, Hugo, or any static generator | `StaticSite` | [Static sites](static-site.md) |

See
[examples/fly-website-vite](https://github.com/alchemy-run/alchemy/tree/main/examples/fly-website-vite)
for a Vite SPA next to a Fly Service API.

## How to choose

Use the resource named after your framework. `Astro`, `Nextjs`,
`Nuxt`, `ReactRouter`, `SolidStart`, `SvelteKit`,
`TanStackStart`, `Waku`, `Octane`, `Foldkit`, and `Vocs` each
drive that framework's programmatic build and know its output
layout, config surface, and dev server.

Use `Vite` when the whole deployable output is static assets — a
React, Vue, or Solid SPA, or an `index.html`-per-route multi-page
app. SSR frameworks that wrap Vite (`Astro`, `SvelteKit`,
`Octane`, …) have their own resources; `Vite` never starts a
framework server module.

Use `StaticSite` when the build is an arbitrary shell command
that emits a directory of files.

## Environment variables

Top-level `env` is copied onto `process.env` before build and
dev, and onto the Machine at deploy. Client inlining is whatever
the framework already does (`VITE_*` for Vite). It is not Worker
bindings and not AWS `server.environment`.

```typescript
const site = yield* Fly.Website.Vite("Web", {
  env: {
    VITE_API_URL: api.url,
  },
});
```

Server code reads `process.env`. Pair a site with
[Postgres](https://alchemy.run/fly/data/postgres) the same way you would any other
Service: put the connection string in `env`, or bind
`ConnectPostgres` on a sibling [Service](https://alchemy.run/fly/compute/services).

## Local development

`alchemy dev` runs the framework's own dev server (native HMR)
instead of deploying. `site.url` is the local address and no Fly
App, Service, IP, or certificate is created. Wrap the site in
`Alchemy.remote()` to deploy the live path even during dev.

## Custom domain

```typescript
const site = yield* Fly.Website.Astro("Web", {
  domain: "app.example.com",
});
```

`domain` is a hostname string. Alchemy requests ACME on the App;
point existing DNS at the App first (v1 does not create records).
`url` becomes `https://app.example.com`.

## Where next

- [The Vite resource](vite.md) — assets-only SPA,
  `assets.notFoundHandling`, env inlining.
- [The StaticSite resource](static-site.md) — build
  commands, SPA/404 handling.
- Framework guides:
  [Astro](astro.md),
  [Next.js](nextjs.md),
  [Nuxt](nuxt.md),
  [React Router](react-router.md),
  [SolidStart](solidstart.md),
  [SvelteKit](sveltekit.md),
  [TanStack Start](tanstack-start.md),
  [Waku](waku.md),
  [Octane](octane.md),
  [Foldkit](foldkit.md),
  [Vocs](vocs.md).
- [Services](https://alchemy.run/fly/compute/services) — the Node process these
  resources deploy.
- [Postgres](https://alchemy.run/fly/data/postgres) — managed Postgres next to a
  site.
- [`Vite` reference](https://alchemy.run/providers/fly/website/vite),
  [`Astro` reference](https://alchemy.run/providers/fly/website/astro),
  [`StaticSite` reference](https://alchemy.run/providers/fly/website/staticsite).
