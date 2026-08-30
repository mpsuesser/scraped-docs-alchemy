---
url: https://alchemy.run/railway/frontend/static-site
title: "Static sites"
description: "Deploy a static site to Railway with Railway.Website.StaticSite — a build command, a container Service."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

`Railway.Website.StaticSite` runs a build command, content-hashes
the output directory, and deploys a Node static-file server on one
[Railway Service](https://alchemy.run/railway/compute/services). Use it when the site
has its own build step — Hugo, Zola, Eleventy, or any custom
pipeline.

For Vite-based projects, prefer
[`Railway.Website.Vite`](vite.md).

## Declare the Website

`command` and `outdir` are required. Declare the site as a
module-level const:

```typescript
// alchemy.run.ts
import * as Railway from "alchemy/Railway";

export const Website = Railway.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
});
```

The build is a [`Command.Build`](https://alchemy.run/providers/command/build) —
memoized, so unchanged inputs skip the rebuild
([details](../../command/memoization.md)). By default every non-gitignored
file in `cwd` (plus the nearest lockfile) is hashed. Narrow it with
`memo: { include: [...] }`, or set `memo: false` to rebuild on every
deploy. `shell` and `timeout` from `Command.Build` apply as well.

## Add it to the Stack

Yield the site from your Stack and return its URL:

```typescript
// alchemy.run.ts
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyStaticSite",
  {
    providers: Railway.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

One `StaticSite` expands into a [Project](https://alchemy.run/railway/compute/projects)
(created if you omit `project`) and a Service from a container
image. The Service listens on port 3000 with healthcheck `/health`.
`site.url` is the generated `https://{name}.up.railway.app` (or
`https://{domain}` when you pass `domain`). `service` and `project`
are undefined under `alchemy dev`.

Pass `project` to put the site in a Project you already declared:

```typescript
export const Site = Railway.Project("Site");

export const Website = Railway.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  project: Site,
});
```

## Building from a subdirectory

`cwd` is the working directory for `command`. `outdir` is relative
to `cwd` (default: process cwd):

```typescript
export const Website = Railway.Website.StaticSite("Web", {
  cwd: "apps/web",
  command: "npm run build",
  outdir: "dist",
});
```

## SPAs and 404 pages

A miss — a request that matches no output file — can be answered two
ways, and they are mutually exclusive:

```typescript
// Single-page app: misses serve index.html with a 200 so the
// client-side router takes over.
export const App = Railway.Website.StaticSite("App", {
  command: "npm run build",
  outdir: "dist",
  spa: true,
});

// Static site: misses return a real 404 status with your error page.
export const Docs = Railway.Website.StaticSite("Docs", {
  command: "hugo --minify",
  outdir: "public",
  errorPage: "404.html",
});
```

## Add environment variables

Top-level `env` is copied onto `process.env` for the build and onto
the Service at deploy. Wrap secrets with `Redacted`. It is not
Worker bindings and not AWS `server.environment`:

```diff lang="typescript"
export const Website = Railway.Website.StaticSite("Website", {
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
process as a long-lived child. No Project or Service is created,
and `site.url` is the local address detected from stdout
(`dev.url` pins it when detection fails). `dev` also accepts `cwd`
and `env` overrides:

```typescript
export const Website = Railway.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  dev: { command: "hugo server" },
});
```

Without `dev.command`, the site still builds and a local static
server serves `outdir` — still no cloud resources. Wrap the site in
`Alchemy.remote()` to deploy the live path even during dev:

```typescript
export const Website = Railway.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
}).pipe(Alchemy.remote());
```

## Custom domain

`domain` is a hostname string. Alchemy attaches a
[`Railway.CustomDomain`](https://alchemy.run/providers/railway/customdomain)
(`targetPort: 3000`) and `url` becomes `https://{domain}` instead of
the generated `*.up.railway.app`. Railway's edge terminates TLS:

```typescript
export const Website = Railway.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  domain: "blog.example.com",
});
```

See [Custom domains](https://alchemy.run/railway/networking) for DNS verification.

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

- [Websites on Railway](websites.md) — the full
  websites surface, including Node SSR frameworks
- [StaticSite reference](https://alchemy.run/providers/railway/website/staticsite) —
  every prop and attribute
- [Railway on Alchemy](../../railway.md) — the Railway provider hub
