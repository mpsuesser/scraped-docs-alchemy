---
url: https://alchemy.run/cloudflare/frontend/frontends
title: "Frontend frameworks"
description: "Deploy Vite, Astro, Next.js, Nuxt, SvelteKit, Waku, or any static build to Cloudflare Workers with first-class Website resources."
access_date: 2026-08-06T07:23:05.654Z
current_date: 2026-08-06T07:23:05.654Z
---

Alchemy deploys frontends to Cloudflare with a family of
`Cloudflare.Website` resources. Each one builds your project
programmatically and deploys it as a Worker — server bundle plus
static assets — with zero configuration files: no `wrangler.json`,
no adapter setup.

- [`Vite`](vite.md) — any pure-Vite app: SPAs and
  Vite-plugin frameworks like TanStack Start, React Router, Vue, and
  SolidStart.
- [`Astro`](astro.md) — Astro sites, server-rendered
  or fully static, with an auto-provisioned session KV namespace.
- [`Nextjs`](nextjs.md) — Next.js apps built through
  the OpenNext pipeline, with writable ISR on KV.
- [`Nuxt`](nuxt.md) — Nuxt apps built through
  nitro's `cloudflare_module` preset; your `nuxt.config.ts` loads
  natively.
- [`SvelteKit`](sveltekit.md) — SvelteKit apps with
  a wrangler-free in-memory Cloudflare adapter.
- [`Waku`](waku.md) — Waku (React Server
  Components) apps.
- [`Octane`](octane.md) — OctaneJS fullstack apps
  built through Octane's own Cloudflare adapter.
- [`StaticSite`](static-site.md) — any build
  command's output directory, for static generators like Zola and
  Hugo.

Every resource returns a plain `Worker`, so everything a Worker
supports applies: `domain`, `env` bindings (KV, R2, Durable Objects,
secrets), `compatibility` flags, and asset routing config. All builds
are memoized by content-hashing the input files — an unchanged
project skips the build and deploy entirely.

## What's supported

| Framework | Resource | Guide |
| --- | --- | --- |
| React / Vite SPA | `Vite` | [Vite SPA](vite-spa.md) |
| TanStack Start (React & Solid) | `Vite` | [TanStack Start](tanstack-start.md) |
| React Router (incl. RSC) | `Vite` | [React Router](react-router.md) |
| Vue | `Vite` | [Vue](vue.md) |
| Foldkit | `Vite` | [Foldkit](foldkit.md) |
| SolidStart / SolidJS SSR | `Vite` | [SolidStart](solidstart.md) |
| Astro | `Astro` | [Astro](astro.md) |
| Next.js | `Nextjs` | [Next.js](nextjs.md) |
| Nuxt | `Nuxt` | [Nuxt](nuxt.md) |
| SvelteKit | `SvelteKit` | [SvelteKit](sveltekit.md) |
| Waku | `Waku` | [Waku](waku.md) |
| OctaneJS (fullstack) | `Octane` | [Octane](octane.md) |
| OctaneJS (SPA) | `Vite` | [Octane](octane.md#octane-spas-use-vite-instead) |
| Zola, Hugo, or any static generator | `StaticSite` | [Static sites](static-site.md) |

Every row is backed by a checked-in example or a live deploy test in
the Alchemy repository. The last row is deliberately open-ended:
StaticSite runs any build command that produces a directory, so any
static generator works the same way.

## How to choose

Use the resource named after your framework. `Astro`, `Nextjs`,
`Nuxt`, `SvelteKit`, `Waku`, and `Octane` each drive their
framework's own programmatic build and know its output layout, config
surface, and dev server.

Use `Vite` when the framework is a plugin in your `vite.config.ts`
and a single `vite build` produces the whole app — TanStack Start,
React Router, Vue, SolidStart, or a plain SPA. See
[what "pure Vite" means](vite.md#what-pure-vite-means).

Use `StaticSite` when the build is an arbitrary shell command that
emits a directory of files — Zola, Hugo, or any other generator
without a dedicated resource.

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
  [Next.js](nextjs.md),
  [Nuxt](nuxt.md),
  [SvelteKit](sveltekit.md),
  [Waku](waku.md),
  [Octane](octane.md).
