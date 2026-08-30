---
url: https://alchemy.run/railway/frontend/websites
title: "Websites"
description: "Deploy Vite, Astro, Next.js, Nuxt, React Router, SolidStart, SvelteKit, TanStack Start, Waku, Octane, Foldkit, Vocs, or any static build to Railway with first-class Website resources."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

Alchemy deploys frontends to Railway with a family of
`Railway.Website` resources. Each one builds your project
programmatically and deploys it as a
[Service](https://alchemy.run/railway/compute/services) from a container image — Node
on port 3000, health check `/health` — with no `railway.toml` and
no adapter to put in the framework config. Your framework's own
config file (`vite.config.ts`, `astro.config.*`, `nuxt.config.ts`,
...) loads natively; Alchemy layers its Node container integration
on top.

Omit `project` and Alchemy creates a
[`Railway.Project`](https://alchemy.run/railway/compute/projects). Pass `project` to
put the site next to other Services in an existing Project.

The live path generates a Dockerfile and uploads it; Railway
builds the image. No GHCR. No Docker daemon on your machine.
`alchemy dev` never creates a Project or Service.

The live URL is the generated `https://{name}.up.railway.app`,
or `https://{domain}` when you pass `domain` (a
[`CustomDomain`](https://alchemy.run/railway/networking#custom-domains) on port
3000).

- [`Vite`](vite.md) — client-only Vite apps
  (React, Vue, Solid SPAs, `index.html` multi-page sites); a tiny
  static-file server serves the `vite build` output.
- [`Astro`](astro.md) — Astro sites, server-rendered
  on Node or fully static; your `astro.config.*` loads natively.
- [`Nextjs`](nextjs.md) — Next.js as a long-running
  Node process (`next build`, then `next({ dev: false })`). Not
  OpenNext.
- [`Nuxt`](nuxt.md) — Nuxt apps through nitro's
  Node server; your `nuxt.config.ts` loads natively. Do not set
  `nitro.preset`.
- [`ReactRouter`](react-router.md) — React Router
  v7 in framework mode, built through your own `vite build`.
- [`SolidStart`](solidstart.md) — SolidStart 2.x
  SSR; Alchemy appends nitro's `node` preset to your `vite build`.
- [`SvelteKit`](sveltekit.md) — SvelteKit SSR plus
  prerendered assets; the Node adapter is injected. Do not set
  `kit.adapter`.
- [`TanStackStart`](tanstack-start.md) — TanStack
  Start (React or Solid), built through your own `vite build`.
- [`Waku`](waku.md) — Waku (React Server
  Components) on Node, with SSG pages baked into the image.
- [`Octane`](octane.md) — OctaneJS SSR. Select
  `node()` from `@alchemy.run/frontend-frameworks/octane/node-adapter`
  in `octane.config.ts`.
- [`Foldkit`](foldkit.md) — client-only Foldkit
  SPA; deep links fall back to `index.html`.
- [`Vocs`](vocs.md) — prerendered Vocs docs;
  extensionless pages (`/about` → `about/index.html`).
- [`StaticSite`](static-site.md) — any build
  command's output directory (Hugo, Zola, Eleventy).

The return is `{ url, service, project }` — `service` and
`project` are `undefined` under `alchemy dev`.

Shared props: `rootDir`, `memo`, `env` (process environment on
the Service — not Worker bindings), `dev`, `domain`, `project`.
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
[examples/railway-website-vite](https://github.com/alchemy-run/alchemy/tree/main/examples/railway-website-vite)
for a Vite SPA in a shared Project.

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
dev, and onto the Railway Service at deploy. Client inlining is
whatever the framework already does (`VITE_*` for Vite). It is
not Worker bindings and not AWS `server.environment`.

```typescript
const site = yield* Railway.Website.Vite("Web", {
  project,
  env: {
    VITE_API_URL: api.url,
  },
});
```

Server code reads `process.env`. Pair a site with
[Postgres](https://alchemy.run/railway/data/postgres) in the same Project the same
way you would any other Service.

## Local development

`alchemy dev` runs the framework's own dev server (native HMR)
instead of deploying. `site.url` is the local address and no
Project or Service is created. Wrap the site in
`Alchemy.remote()` to deploy the live path even during dev.

## Custom domain

```typescript
const site = yield* Railway.Website.Astro("Web", {
  project,
  domain: "app.example.com",
});
```

`domain` attaches a `Railway.CustomDomain` (`targetPort: 3000`).
`url` becomes `https://app.example.com` instead of the generated
`*.up.railway.app`.

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
- [Services](https://alchemy.run/railway/compute/services) — the container these
  resources deploy.
- [Custom domains](https://alchemy.run/railway/networking#custom-domains) — hostname
  on a Service.
- [Postgres](https://alchemy.run/railway/data/postgres) — Postgres in the same
  Project.
- [`Vite` reference](https://alchemy.run/providers/railway/website/vite),
  [`Astro` reference](https://alchemy.run/providers/railway/website/astro),
  [`StaticSite` reference](https://alchemy.run/providers/railway/website/staticsite).
