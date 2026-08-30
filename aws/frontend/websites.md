---
url: https://alchemy.run/aws/frontend/websites
title: "Websites"
description: "Deploy Vite, Astro, Next.js, Nuxt, React Router, SvelteKit, TanStack Start, Waku, Octane, or any static build to AWS with first-class Website resources."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

Alchemy deploys frontends to AWS with a family of `AWS.Website`
resources. Each one builds your project programmatically and deploys
it as a serverless site — the SSR server on a streaming Lambda
Function URL, static assets in a private S3 bucket, and a CloudFront
distribution whose edge router serves uploaded files from S3 and
forwards everything else to the server — with no AWS-specific
configuration: no CDK, no CloudFormation, no adapter setup. Your
framework's own config file (`astro.config.*`, `nuxt.config.ts`,
`vite.config.ts`, ...) loads natively; Alchemy layers its AWS
integration on top.

- [`Vite`](vite.md) — any client-only Vite project (React,
  Vue, Solid, or a plain SPA): assets-only, no Lambda; your
  `vite.config.*` loads natively.
- [`Foldkit`](foldkit.md) — Foldkit apps; `Vite` with SPA
  deep links on by default.
- [`Astro`](astro.md) — Astro sites, server-rendered or
  fully static; your `astro.config.*` loads natively.
- [`Nextjs`](nextjs.md) — Next.js apps built through the
  OpenNext (`@opennextjs/aws`) pipeline, with streaming SSR, image
  optimization, and ISR wiring; your `next.config.*` is honored
  as-is.
- [`Nuxt`](nuxt.md) — Nuxt apps built through nitro's
  `aws-lambda` preset; your `nuxt.config.ts` loads natively.
- [`ReactRouter`](react-router.md) — React Router v7 in
  framework mode, built through your own `vite build`.
- [`SvelteKit`](sveltekit.md) — SvelteKit apps with a
  wrangler-free in-memory AWS adapter; your `vite.config.ts` loads
  natively.
- [`TanStackStart`](tanstack-start.md) — TanStack Start
  (React or Solid), built through your own `vite build`.
- [`Waku`](waku.md) — Waku (React Server Components)
  apps.
- [`Octane`](octane.md) — OctaneJS fullstack apps built
  through your own `vite build` with the AWS marker adapter.
- [`StaticSite`](static-site.md) — any directory of files,
  optionally produced by a build command, for static generators like
  Zola and Hugo or pre-built SPAs.
- [`Router`](static-site.md#compose-sites-with-a-router) —
  a shared CloudFront front door: one distribution serving several
  sites (or a site plus an API), routed at the edge.

Every resource shares the same surface: `domain` (ACM certificate +
Route 53 records), server configuration (`memorySize`, `timeout`,
`env`), `assets`, `edge` customizations, and `invalidation`.
All builds are memoized by content-hashing the input files — an
unchanged project skips the build and deploy entirely. Under
`alchemy dev`, every resource runs the framework's own dev server
(native HMR) instead of deploying; `Alchemy.remote()` opts back into
the full live deployment.

## What's supported

| Framework | Resource | Guide |
| --- | --- | --- |
| React / Vite SPA | `Vite` | [React SPA](vite-spa.md) |
| Vue | `Vite` | [Vue](vue.md) |
| Foldkit | `Foldkit` | [Foldkit](foldkit.md) |
| TanStack Start (React & Solid) | `TanStackStart` | [TanStack Start](tanstack-start.md) |
| React Router | `ReactRouter` | [React Router](react-router.md) |
| Astro | `Astro` | [Astro](astro.md) |
| Next.js | `Nextjs` | [Next.js](nextjs.md) |
| Nuxt | `Nuxt` | [Nuxt](nuxt.md) |
| SvelteKit | `SvelteKit` | [SvelteKit](sveltekit.md) |
| Waku | `Waku` | [Waku](waku.md) |
| OctaneJS (fullstack) | `Octane` | [Octane](octane.md) |
| Zola, Hugo, or any static generator | `StaticSite` | [Static sites](static-site.md) |

Every row is backed by a checked-in example or a live deploy test in
the Alchemy repository. The last row is deliberately open-ended:
`StaticSite` deploys any directory of files (running your build
command first if you give it one), so any generator works the same
way.

## How to choose

Use the resource named after your framework. `Astro`, `Nextjs`,
`Nuxt`, `ReactRouter`, `SvelteKit`, `TanStackStart`,
`Waku`, and `Octane` each drive their framework's own build and know
its output layout, config surface, and dev server.

Use `Vite` when the app is client-only and a single `vite build`
produces the whole thing — a React or Vue SPA, or any Vite project
with no server. It creates no Lambda.

Use `StaticSite` when the output is plain files from an arbitrary
build command that emits a directory (Zola, Hugo, or any other
generator without a dedicated resource).

Use `Router` when several sites (or a site plus an API) should share
one CloudFront distribution and one domain — distributions take
minutes to create, and each custom domain can only attach to one.

## Where next

- [The Vite resource](vite.md) — build model, env
  inlining, SPA fallback, dev mode.
- [The StaticSite resource](static-site.md) — build
  commands, SPA/404 handling, Router composition, invalidation.
- Framework guides:
  [React SPA](vite-spa.md),
  [Vue](vue.md),
  [Foldkit](foldkit.md),
  [TanStack Start](tanstack-start.md),
  [React Router](react-router.md),
  [Astro](astro.md),
  [Next.js](nextjs.md),
  [Nuxt](nuxt.md),
  [SvelteKit](sveltekit.md),
  [Waku](waku.md),
  [Octane](octane.md).
- [Full-stack RPC + Drizzle](full-stack-tanstack-rpc-drizzle.md) —
  a TanStack Start UI driving an Effect RPC Lambda over Aurora DSQL.
- [`StaticSite` reference](https://alchemy.run/providers/aws/website/staticsite) and
  [`Router` reference](https://alchemy.run/providers/aws/website/router) — every prop
  and attribute.
