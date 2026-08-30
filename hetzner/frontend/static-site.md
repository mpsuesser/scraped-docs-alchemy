---
url: https://alchemy.run/hetzner/frontend/static-site
title: "Static sites"
description: "Deploy a static site to Hetzner with Hetzner.Website.StaticSite — a build command, a systemd unit on a Server, and http://{ipv4}:3000 (or your hostname) on deploy."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

`Hetzner.Website.StaticSite` runs a build command, packs the output
directory into the unit archive, and deploys a tiny static-file
server (`GET` assets, `/health`, optional SPA / 404-page) as a
[Service](https://alchemy.run/hetzner/compute/services) on a
[Server](https://alchemy.run/hetzner/compute/servers). Use it when the site has its own
build step — Hugo, Zola, Eleventy, or any custom pipeline.

For Vite-based projects, prefer
[`Hetzner.Website.Vite`](vite.md).

## Declare the Website

`command` and `outdir` are required. Declare the site as a
module-level const:

```typescript
// alchemy.run.ts
import * as Hetzner from "alchemy/Hetzner";

export const Website = Hetzner.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
});
```

The build is a [`Command.Build`](https://alchemy.run/providers/command/build) —
memoized, so unchanged inputs skip the rebuild
([details](../../command/memoization.md)). By default every non-gitignored
file in `cwd` (plus the nearest lockfile) is hashed. Narrow it with
`memo: { include: [...] }`, or set `memo: false` to rebuild on every
deploy.

## Add it to the Stack

Yield the site from your Stack and return its URL:

```typescript
// alchemy.run.ts
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyStaticSite",
  {
    providers: Hetzner.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

One `StaticSite` expands into a Server (default `cpx12` /
`ubuntu-24.04` / `fsn1` if you omit `server`) and a systemd unit on
port 3000. Several sites or APIs can share one Server. Live `url` is
`http://{ipv4}:3000` (or `http://{domain}:3000` when you pass
`domain`). There is no TLS on the Service. `server` and `service`
are undefined under `alchemy dev`.

Pass `server` to colocate the unit with other Services:

```typescript
export const Box = Hetzner.Server("Box", {
  serverType: "cpx12",
  image: "ubuntu-24.04",
  location: "fsn1",
});

export const Website = Hetzner.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  server: Box,
});
```

## Building from a subdirectory

`cwd` is the working directory for `command`. `outdir` is relative
to `cwd` (default: process cwd):

```typescript
export const Website = Hetzner.Website.StaticSite("Web", {
  cwd: "apps/web",
  command: "npm run build",
  outdir: "dist",
});
```

## SPAs and 404 pages

A miss — a request that matches no output file — can be answered two
ways, and they are mutually exclusive (passing both fails the
deploy):

```typescript
// Single-page app: misses serve index.html with a 200 so the
// client-side router takes over.
export const App = Hetzner.Website.StaticSite("App", {
  command: "npm run build",
  outdir: "dist",
  spa: true,
});

// Static site: misses return a real 404 status with your error page.
export const Docs = Hetzner.Website.StaticSite("Docs", {
  command: "hugo --minify",
  outdir: "public",
  errorPage: "404.html",
});
```

## Add environment variables

Top-level `env` is copied onto `process.env` for the build and onto
the systemd unit at deploy. Wrap secrets with `Redacted`. It is not
Worker bindings and not AWS `server.environment`:

```diff lang="typescript"
export const Website = Hetzner.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
+  env: {
+    HUGO_ENV: "production",
+  },
});
```

Generators that inline at build time pick these up. The hosted
process is the generated static-file server — there is no framework
SSR runtime.

## Local development

During `alchemy dev`, `dev.command` replaces the build with that
process as a long-lived child. No Server or Service is created, and
`site.url` is the local address detected from stdout (`dev.url`
pins it when detection fails). `dev` also accepts `cwd` and `env`
overrides:

```typescript
export const Website = Hetzner.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  dev: { command: "hugo server" },
});
```

Without `dev.command`, the site still builds and a local static
server serves `outdir` — still no cloud resources. Wrap the site in
`Alchemy.remote()` to deploy the live path even during dev:

```typescript
export const Website = Hetzner.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
}).pipe(Alchemy.remote());
```

## Custom domain

`domain` requires `zone` — a
[`Hetzner.Zone`](https://alchemy.run/hetzner/networking/dns) you pass in. The site
does not create one. Alchemy creates an A `RecordSet` pointing at
the Server's public IPv4, and `url` becomes `http://{domain}:3000`.
The Service still has no TLS:

```typescript
export const Dns = Hetzner.Zone("Dns", {
  name: "example.com",
});

export const Website = Hetzner.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  domain: "blog.example.com",
  zone: Dns,
});
```

Passing `domain` without `zone` fails the deploy.

## When to use a framework resource instead

`StaticSite` is the general fallback for any shell command that
produces a directory of files. Frameworks with dedicated resources
have a better path: [Astro](astro.md),
[Foldkit](foldkit.md),
[Next.js](nextjs.md), [Nuxt](nuxt.md),
[Octane](octane.md),
[React Router](react-router.md),
[SolidStart](solidstart.md),
[SvelteKit](sveltekit.md),
[TanStack Start](tanstack-start.md),
[Vite](vite.md), [Vocs](vocs.md), and
[Waku](waku.md). Those resources run the framework's
own programmatic build and skip the `command` / `outdir` contract.
Reach for `StaticSite` when there is no dedicated resource — Zola,
Hugo, or any other generator.

## Where next

- [Websites on Hetzner](websites.md) — the full
  websites surface, including Node SSR frameworks
- [StaticSite reference](https://alchemy.run/providers/hetzner/website/staticsite) —
  every prop and attribute
- [Hetzner on Alchemy](../../hetzner.md) — the Hetzner provider hub
