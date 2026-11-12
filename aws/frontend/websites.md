---
url: https://alchemy.run/aws/frontend/websites
title: "Websites"
description: "Deploy Vite, Astro, Next.js, vinext, Nuxt, React Router, SvelteKit, TanStack Start, Waku, Octane, or any static build to AWS with first-class Website resources."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A website on AWS is three pieces working together. Its static files, the scripts, styles and images, sit in a private S3 Bucket. Its server, if it has one, runs on a [Lambda Web Function](../compute/web-functions.md). And a CloudFront distribution in front of both serves every request from a location near the visitor: files straight from the Bucket, everything else from the server.

<svg viewBox="0 0 738 204" style="max-width: 849px" role="img" aria-label="CloudFront serves files from the Bucket and sends the rest to the server."><defs><marker id="alc-dag-arrow-l3mu9a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 172 102 C 230.5 102, 230.5 102, 289 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-l3mu9a)"></path></g><g><path d="M 443 102 C 501.5 102, 501.5 56, 560 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-l3mu9a)"></path><text fill="currentColor" x="501.5" y="73" text-anchor="middle">/assets/*</text></g> <g><path d="M 443 102 C 501.5 102, 501.5 148, 560 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-l3mu9a)"></path><text fill="currentColor" x="501.5" y="119" text-anchor="middle">everything else</text></g> <g transform="translate(24, 70)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">visitor</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">app.example.com</text></g> <g transform="translate(295, 70)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">CloudFront</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">edge router</text></g> <g transform="translate(566, 24)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Assets</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">S3 Bucket</text></g> <g transform="translate(566, 116)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Server</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">Lambda Web Function</text></g></svg> <svg viewBox="0 0 328 332" style="max-width: 377px" role="img" aria-label="CloudFront serves files from the Bucket and sends the rest to the server."><defs><marker id="alc-dag-arrow-ghtbgj" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 74 C 164 101, 164 101, 164 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ghtbgj)"></path></g><g><path d="M 164 198 C 164 225, 80 225, 80 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ghtbgj)"></path><text fill="currentColor" x="122" y="219" text-anchor="middle">/assets/*</text></g> <g><path d="M 164 198 C 164 225, 248 225, 248 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ghtbgj)"></path><text fill="currentColor" x="206" y="219" text-anchor="middle">everything else</text></g> <g transform="translate(94, 10)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">visitor</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">app.example.com</text></g> <g transform="translate(94, 134)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">CloudFront</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">edge router</text></g> <g transform="translate(10, 258)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Assets</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">S3 Bucket</text></g> <g transform="translate(178, 258)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Server</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Lambda Web Function</text></g></svg>

CloudFront serves files from the Bucket and sends the rest to the server.

Each framework has a Website resource that builds your project and creates all three. Your framework’s own config file loads as it is, with nothing AWS-specific in it:

```typescript
const site = yield* AWS.Website.Astro("Website", { domain: "app.example.com" });
```

Every Website resource takes the same props: a `domain`, with its certificate and DNS records (see [Custom domains](../networking/custom-domains.md)), the server’s `timeout`, `env` and scaling, and how assets are uploaded and cached. A build is skipped when no input file has changed. Under `alchemy dev`, each one runs the framework’s own dev server instead of deploying.

A site with no server, a single-page app or a static generator’s output, creates no Function at all: just the Bucket and the distribution.

## Serve from several Regions

List extra Regions to run the server in each of them behind one global domain. CloudFront forwards to it, and each request reaches the nearest Region:

```typescript
const site = yield* AWS.Website.Nuxt("Website", {
  regions: ["us-east-1", "eu-west-1"],
});
```

The site’s URL doesn’t change when you add or remove Regions. `Nextjs` and `Vinext` keep their data caches in the home Region, so a server in another Region reaches back to it for cached pages; `regions` helps most for sites without a server-side cache.

## Choose the server

The SSR server runs on a Lambda Web Function: your framework’s own Node server, serving many requests per environment and streaming each response. Web Functions run in a subset of Regions. Elsewhere, set `server: "function"` to run the server on a Lambda Function behind a streaming Function URL:

```typescript
const site = yield* AWS.Website.Nuxt("Website", {
  server: "function",
  memorySize: 2048,
});
```

`memorySize`, `architecture`, and `runtime` apply only to `server: "function"`; a Web Function has a fixed 2 GB, Arm64 and Node.js 24.

## What’s supported

| Framework | Resource | Guide |
| --- | --- | --- |
| React / Vite SPA | `Vite` | [React SPA](vite-spa.md) |
| Vue | `Vite` | [Vue](vue.md) |
| Foldkit | `Foldkit` | [Foldkit](foldkit.md) |
| TanStack Start (React & Solid) | `TanStackStart` | [TanStack Start](tanstack-start.md) |
| React Router | `ReactRouter` | [React Router](react-router.md) |
| Astro | `Astro` | [Astro](astro.md) |
| Next.js | `Nextjs` | [Next.js](nextjs.md) |
| vinext | `Vinext` | [vinext](vinext.md) |
| Nuxt | `Nuxt` | [Nuxt](nuxt.md) |
| SvelteKit | `SvelteKit` | [SvelteKit](sveltekit.md) |
| Waku | `Waku` | [Waku](waku.md) |
| OctaneJS (fullstack) | `Octane` | [Octane](octane.md) |
| Zola, Hugo, or any static generator | `StaticSite` | [Static sites](static-site.md) |

Every row is backed by a checked-in example or a live deploy test in the Alchemy repository. The last row is deliberately open-ended: `StaticSite` deploys any directory of files (running your build command first if you give it one), so any generator works the same way.

## How to choose

Use the resource named after your framework. `Astro`, `Nextjs`, `Nuxt`, `ReactRouter`, `SvelteKit`, `TanStackStart`, `Waku`, and `Octane` each drive their framework’s own build and know its output layout, config surface, and dev server.

Use `Vite` when the app is client-only and a single `vite build` produces the whole thing — a React or Vue SPA, or any Vite project with no server. It creates no Lambda.

Use `StaticSite` when the output is plain files from an arbitrary build command that emits a directory (Zola, Hugo, or any other generator without a dedicated resource).

Use `Router` when several sites (or a site plus an API) should share one CloudFront distribution and one domain — distributions take minutes to create, and each custom domain can only attach to one.

## Where next

- [The Vite resource](vite.md) — build model, env inlining, SPA fallback, dev mode.
- [The StaticSite resource](static-site.md) — build commands, SPA/404 handling, Router composition, invalidation.
- Framework guides: [React SPA](vite-spa.md), [Vue](vue.md), [Foldkit](foldkit.md), [TanStack Start](tanstack-start.md), [React Router](react-router.md), [Astro](astro.md), [Next.js](nextjs.md), [Nuxt](nuxt.md), [SvelteKit](sveltekit.md), [Waku](waku.md), [Octane](octane.md).
- [Full-stack RPC + Drizzle](full-stack-tanstack-rpc-drizzle.md) — a TanStack Start UI driving an Effect RPC Lambda over Aurora DSQL.
- [`StaticSite` reference](https://alchemy.run/providers/aws/website#staticsite) and [`Router` reference](https://alchemy.run/providers/aws/website#router) — every prop and attribute.
