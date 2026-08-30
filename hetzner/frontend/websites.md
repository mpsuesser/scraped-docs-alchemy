---
url: https://alchemy.run/hetzner/frontend/websites
title: "Websites"
description: "Deploy Vite, Astro, Next.js, Nuxt, React Router, SolidStart, SvelteKit, TanStack Start, Waku, Octane, Foldkit, Vocs, or any static build to Hetzner with first-class Website resources."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

Alchemy deploys frontends to Hetzner with a family of
`Hetzner.Website` resources. Each one builds your project
programmatically and deploys it as a systemd
[Service](https://alchemy.run/hetzner/compute/services) on a
[Server](https://alchemy.run/hetzner/compute/servers) — Node on port 3000, no
Dockerfile, no unit file, no adapter in the framework config.
Your framework's own config file (`vite.config.ts`,
`astro.config.*`, `nuxt.config.ts`, ...) loads natively; Alchemy
layers its Node server on top.

Omit `server` and Alchemy creates a `cpx12` / `ubuntu-24.04`
Server in `fsn1`. Pass `server` to put several sites (or a site
plus an API) on one VM.

The live URL is **`http://{ipv4}:3000`**. There is no TLS on the
Service. `domain` requires an existing Hetzner DNS
[`Zone`](https://alchemy.run/hetzner/networking/dns) and creates an A record;
`url` becomes `http://{domain}:3000`.

- [`Vite`](vite.md) — client-only Vite apps
  (React, Vue, Solid SPAs, `index.html` multi-page sites); a
  static-file unit serves the `vite build` output.
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
  Components) on Node, with SSG pages served extensionless.
- [`Octane`](octane.md) — OctaneJS SSR. Select
  `node()` from `@alchemy.run/frontend-frameworks/octane/node-adapter`
  in `octane.config.ts`.
- [`Foldkit`](foldkit.md) — client-only Foldkit
  SPA; deep links fall back to `index.html`.
- [`Vocs`](vocs.md) — prerendered Vocs docs;
  extensionless pages (`/about` → `about/index.html`).
- [`StaticSite`](static-site.md) — any build
  command's output directory (Hugo, Zola, Eleventy).

The return is `{ url, server, service }` — `server` and
`service` are `undefined` under `alchemy dev`.

Shared props: `rootDir`, `memo`, `env` (process environment on
the unit — not Worker bindings), `dev`, `domain`, `zone`,
`server`. Builds are memoized by content-hashing the input
files — an unchanged project skips the build.

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
[examples/hetzner-website-vite](https://github.com/alchemy-run/alchemy/tree/main/examples/hetzner-website-vite)
for a Vite SPA sharing a Server with an API unit.

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

Pass one `Hetzner.Server` into several Website (and API)
resources when they should share a VM.

## Environment variables

Top-level `env` is copied onto `process.env` before build and
dev, and onto the systemd unit at deploy. Client inlining is
whatever the framework already does (`VITE_*` for Vite). It is
not Worker bindings and not AWS `server.environment`.

```typescript
const site = yield* Hetzner.Website.Vite("Web", {
  server,
  env: {
    VITE_API_URL: api.url,
  },
});
```

Server code reads `process.env`.

## Local development

`alchemy dev` runs the framework's own dev server (native HMR)
instead of deploying. `site.url` is the local address and no
Server or Service is created. Wrap the site in
`Alchemy.remote()` to deploy the live path even during dev.

## Custom domain

`domain` requires `zone` — an existing Hetzner DNS Zone. v1
does not create a Zone. Alchemy adds an A
[`RecordSet`](https://alchemy.run/hetzner/networking/dns) pointing at the Server's
public IPv4. Passing `domain` without `zone` fails.

```typescript
const site = yield* Hetzner.Website.Astro("Web", {
  server,
  domain: "app.example.com",
  zone,
});
```

`url` is `http://app.example.com:3000`. The Service itself does
not terminate TLS.

## Where next

- [The Vite resource](vite.md) — assets-only SPA,
  `assets.notFoundHandling`, sharing a Server.
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
- [Servers](https://alchemy.run/hetzner/compute/servers) and
  [Services](https://alchemy.run/hetzner/compute/services) — the VM and systemd
  unit these resources deploy.
- [Zones & records](https://alchemy.run/hetzner/networking/dns) — the Zone
  `domain` attaches to.
- [`Vite` reference](https://alchemy.run/providers/hetzner/website/vite),
  [`Astro` reference](https://alchemy.run/providers/hetzner/website/astro),
  [`StaticSite` reference](https://alchemy.run/providers/hetzner/website/staticsite).
