---
url: https://alchemy.run/cloudflare/frontend/frontends
title: "Frontend frameworks"
description: "Deploy any pure-Vite app (TanStack Start, React Router, Vue, SolidStart) with Cloudflare.Website.Vite, or any build command's static output (Zola, Astro) with Cloudflare.Website.StaticSite."
access_date: 2026-08-03T19:38:24.228Z
current_date: 2026-08-03T19:38:24.228Z
---

Alchemy deploys frontends to Cloudflare with two resources:
`Cloudflare.Website.Vite` for anything built by Vite, and
`Cloudflare.Website.StaticSite` for anything built by any other
command.

## Cloudflare.Website.Vite

```typescript
const site = yield* Cloudflare.Website.Vite("Website");
```

`Cloudflare.Website.Vite` deploys **any pure-Vite app** — it runs a
programmatic `vite build` of your project with the Cloudflare plugin
appended to the plugins your own `vite.config.ts` declares, then
uploads the client assets (and server bundle, if the build produces
one) as a Worker. No `main` entrypoint, no build command, no output
directory, no Wrangler config. Whatever framework plugin drives your
Vite build — TanStack Start, React Router, Vue, SolidStart — rides
along unchanged.

## Cloudflare.Website.StaticSite

```typescript
const site = yield* Cloudflare.Website.StaticSite("Website", {
  command: "zola build",
  outdir: "public",
});
```

`Cloudflare.Website.StaticSite` deploys **any directory produced by
any build command** — it runs the command as a shell process,
content-hashes the output directory, and serves it as Worker static
assets. The framework (or lack of one) doesn't matter: if a command
produces a directory of files, StaticSite deploys it.

Both resources are thin wrappers over a
[Worker](https://alchemy.run/providers/cloudflare/workers/worker), so everything a Worker
supports applies to both: `domain`, `env` bindings, `compatibility`
flags, and a custom `main` entrypoint in front of the assets.

## What's supported

| Framework | Resource | Status | Guide |
| --- | --- | --- | --- |
| React / Vite SPA | `Vite` | Supported | [Vite SPA](vite-spa.md) |
| TanStack Start (React & Solid) | `Vite` | Supported | [TanStack Start](tanstack-start.md) |
| React Router (incl. RSC) | `Vite` | Supported | [React Router](react-router.md) |
| Vue | `Vite` | Supported | [Vue](vue.md) |
| Foldkit | `Vite` | Supported | [Foldkit](foldkit.md) |
| SolidStart / SolidJS SSR | `Vite` | Supported | [SolidStart](solidstart.md) |
| Astro | `StaticSite` (static output) | Vite support is a TODO | [Astro](astro.md) |
| Nuxt | — | Not yet supported | [Nuxt](nuxt.md) |
| Zola, Hugo, or any static generator | `StaticSite` | Supported | [Static sites](static-site.md) |

Every "Supported" row is backed by a checked-in example or a live
deploy test in the Alchemy repository — plain Vite SPA deploys are
live-tested with a vanilla fixture, and React itself is exercised
via the TanStack Start example. The last row is deliberately
open-ended: StaticSite runs any build command that produces a
directory, so any static generator works the same way — Zola is the
one exercised as an example.

## How to choose

The criterion is literal: `Cloudflare.Website.Vite` calls
`vite.createBuilder({ root: rootDir, plugins: [cloudflare(...), ...] })`
on your project root and runs `buildApp()` — nothing else.

If your **entire app is built by `vite build`** — the framework is a
plugin declared in `vite.config.ts` (`tanstackStart()`, `viteReact()`,
`@vitejs/plugin-vue`, `solidStart()`) — use `Vite`. If the framework
drives its own CLI build (`astro build`, `nuxi build`) or isn't
JavaScript at all (Zola, Hugo), that build never enters Vite's
builder, so use `StaticSite` on the build output instead. See
[what "pure Vite" means](vite.md#what-pure-vite-means)
for the full picture.

## Where next

- [The Vite resource](vite.md) — build model, env
  inlining, runtime bindings, dev mode.
- [The StaticSite resource](static-site.md) — build
  commands, custom Workers in front of assets, framework-native dev
  servers.
- Framework guides:
  [Vite SPA](vite-spa.md),
  [TanStack Start](tanstack-start.md),
  [React Router](react-router.md),
  [Vue](vue.md),
  [SolidStart](solidstart.md),
  [Astro](astro.md),
  [Nuxt](nuxt.md).
