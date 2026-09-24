---
url: https://alchemy.run/prisma/frontend/static-site
title: "Static sites"
description: "Deploy any static build to Prisma Compute with Prisma.Website.StaticSite — a shell command, output directory, and Bun static-file server."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

`Prisma.Website.StaticSite` runs a build command, packages its output,
and deploys a static-file server on **Bun in Prisma Compute**. Use it
for Hugo, Zola, Eleventy, or a custom pipeline. The output is uploaded
as `tar.gz`; there is no Docker image or registry.

Static sites still run on Compute. This resource does not publish files
to a global CDN or remove the runtime. For Vite apps, prefer
[`Prisma.Website.Vite`](vite.md).

## Declare the Website

`command` and `outdir` are required:

```typescript
// alchemy.run.ts
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
});
```

Install the generator your command invokes. Unlike framework resources,
`StaticSite` does not load a framework build integration from your app;
it uses [`Command.Build`](https://alchemy.run/providers/command/reference#build) and the shared
static-file server.

The command is memoized from input content. Use `memo: { include: [...] }`
to narrow inputs or `memo: false` to rebuild every time. `shell` and
`timeout` follow `Command.Build`; see [Memoization](../../command/memoization.md).

## Add it to the Stack

```typescript
// alchemy.run.ts
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyStaticSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

Live deploy creates a database-less Prisma project if you omit `project`.
Pass an existing project to share it with other apps. `site.url` is the
Compute endpoint, or the HTTPS hostname when `domain` is supplied.

## Building from a subdirectory

`cwd` is the command's working directory; it defaults to `rootDir`, then
`.`. `outdir` is relative to that directory:

```typescript
export const Website = Prisma.Website.StaticSite("Web", {
  rootDir: "apps/web",
  command: "npm run build",
  outdir: "dist",
});
```

Use a dedicated build directory, not the application root. Only the
output files are packaged, not the whole source repository.

## SPAs and 404 pages

A client-side router needs the index page on an unmatched URL:

```typescript
export const App = Prisma.Website.StaticSite("App", {
  command: "npm run build",
  outdir: "dist",
  assets: { notFoundHandling: "single-page-application" },
});
```

This returns `index.html` with status 200. `spa: true` is the equivalent
command-site shorthand.

A static site can return a custom error page with status 404:

```typescript
export const Docs = Prisma.Website.StaticSite("Docs", {
  command: "hugo --minify",
  outdir: "public",
  errorPage: "404.html",
});
```

`errorPage` is relative to `outdir`; it is mutually exclusive with
`spa: true`. For the conventional `404.html`,
`assets: { notFoundHandling: "404-page" }` also works.

## Add environment variables

```diff lang="typescript"
export const Website = Prisma.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
+  env: { HUGO_ENV: "production" },
});
```

`env` is passed to the build and Compute. Generators may embed values
in their output; `Redacted` protects display, not generated public files.
The hosted process is the static-file server, not a framework SSR handler.

## Local development

Supply the generator's native server to skip the production build:

```typescript
export const Website = Prisma.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  dev: { command: "hugo server" },
});
```

`dev.cwd` and `dev.env` override the command's directory and environment.
The local URL is detected from stdout; set `dev.url` if the command does
not print one.

Without `dev.command`, Alchemy builds and serves `outdir` locally.
Neither local path creates a Prisma project, database, app, or domain.
Opt into live deployment during dev explicitly:

```typescript
export const Website = Prisma.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
}).pipe(Alchemy.remote());
```

## Custom domain

```typescript
export const Website = Prisma.Website.StaticSite("Website", {
  command: "hugo --minify",
  outdir: "public",
  domain: "blog.example.com",
});
```

`domain` creates `Prisma.CustomDomain` and changes `site.url` to the
HTTPS hostname. The app must be on the project's current default branch.
Configure its returned DNS records yourself and check status before
cutover; see [Custom domains](websites.md#custom-domains).

## When to use a framework resource instead

[Astro](astro.md), [Foldkit](foldkit.md),
[Next.js](nextjs.md), [Nuxt](nuxt.md),
[Octane](octane.md), [React Router](react-router.md),
[SolidStart](solidstart.md), [SvelteKit](sveltekit.md),
[TanStack Start](tanstack-start.md), [Vite](vite.md),
[Vocs](vocs.md), and [Waku](waku.md) each know
their own programmatic build and output layout. Use those instead of
specifying `command` and `outdir` when your framework has a resource.

## Where next

- [StaticSite API](https://alchemy.run/providers/prisma/website#staticsite).
- [Static-site example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-static).
- [Websites](websites.md) and [Compute apps](../compute/apps.md).
