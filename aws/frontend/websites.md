---
url: https://alchemy.run/aws/frontend/websites
title: "Websites"
description: "Deploy Astro, Next.js, Nuxt, SvelteKit, Waku, Octane, or any static build to AWS with first-class Website resources."
access_date: 2026-08-21T19:05:43.655Z
current_date: 2026-08-21T19:05:43.655Z
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

- [`Astro`](astro.md) — Astro sites, server-rendered or
  fully static; your `astro.config.*` loads natively.
- [`Nextjs`](nextjs.md) — Next.js apps built through the
  OpenNext (`@opennextjs/aws`) pipeline, with streaming SSR, image
  optimization, and ISR wiring; your `next.config.*` is honored
  as-is.
- [`Nuxt`](nuxt.md) — Nuxt apps built through nitro's
  `aws-lambda` preset; your `nuxt.config.ts` loads natively.
- [`SvelteKit`](sveltekit.md) — SvelteKit apps with a
  wrangler-free in-memory AWS adapter; your `vite.config.ts` loads
  natively.
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
Route 53 records), `server` configuration (memory, timeout,
environment), `assets`, `edge` customizations, and `invalidation`.
All builds are memoized by content-hashing the input files — an
unchanged project skips the build and deploy entirely. Under
`alchemy dev`, every resource runs the framework's own dev server
(native HMR) instead of deploying; `Alchemy.remote()` opts back into
the full live deployment.

## What's supported

| Framework | Resource | Guide |
| --- | --- | --- |
| Astro | `Astro` | [Astro](astro.md) |
| Next.js | `Nextjs` | [Next.js](nextjs.md) |
| Nuxt | `Nuxt` | [Nuxt](nuxt.md) |
| SvelteKit | `SvelteKit` | [SvelteKit](sveltekit.md) |
| Waku | `Waku` | [Waku](waku.md) |
| OctaneJS (fullstack) | `Octane` | [Octane](octane.md) |
| Vite SPA (any build) | `StaticSite` | [Static sites](static-site.md) |
| Zola, Hugo, or any static generator | `StaticSite` | [Static sites](static-site.md) |

Every row is backed by a checked-in example or a live deploy test in
the Alchemy repository. The last rows are deliberately open-ended:
`StaticSite` deploys any directory of files (running your build
command first if you give it one), so any generator works the same
way.

## How to choose

Use the resource named after your framework. `Astro`, `Nextjs`,
`Nuxt`, `SvelteKit`, `Waku`, and `Octane` each drive their
framework's own programmatic build and know its output layout, config
surface, and dev server.

Use `StaticSite` when the output is plain files — a pre-built SPA, or
an arbitrary build command that emits a directory (Zola, Hugo, or any
other generator without a dedicated resource).

Use `Router` when several sites (or a site plus an API) should share
one CloudFront distribution and one domain — distributions take
minutes to create, and each custom domain can only attach to one.

## Where next

- [The StaticSite resource](static-site.md) — build
  commands, SPA/404 handling, Router composition, invalidation.
- Framework guides:
  [Astro](astro.md),
  [Next.js](nextjs.md),
  [Nuxt](nuxt.md),
  [SvelteKit](sveltekit.md),
  [Waku](waku.md),
  [Octane](octane.md).
- [`StaticSite` reference](https://alchemy.run/providers/aws/website/staticsite) and
  [`Router` reference](https://alchemy.run/providers/aws/website/router) — every prop
  and attribute.
