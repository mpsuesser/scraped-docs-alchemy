---
url: https://alchemy.run/cloudflare/frontend/vite
title: "Vite"
description: "Deploy any pure-Vite app to Cloudflare Workers with a single resource."
access_date: 2026-08-06T07:23:05.654Z
current_date: 2026-08-06T07:23:05.654Z
---

`Cloudflare.Website.Vite` deploys any app that is pure Vite — a project whose entire build is `vite build` with plugins from your `vite.config.ts`. That covers a plain `index.html`, a React or Vue SPA, and full-stack SSR frameworks like TanStack Start or React Router. This page explains the resource itself; each supported framework has [its own landing page](#frameworks) that flows from here.

## One resource, one build

The minimal form takes no props at all:

```typescript
const site = yield* Cloudflare.Website.Vite("Website");
```

Alchemy loads **your** project’s Vite install (resolved from `rootDir` ’s `package.json`), appends Alchemy’s own Cloudflare Vite plugin to the plugins your `vite.config.ts` already declares, and runs `builder.buildApp()`. Client assets are uploaded as Worker static assets; if the build emits a server bundle (SSR), that bundle becomes the Worker entry — otherwise the Worker is assets-only. There is no `main` entrypoint, no build command, no output directory, and no `wrangler.jsonc`: an `index.html` next to your `alchemy.run.ts` (plus your `vite.config.ts`, if you have one) is enough.

## What pure Vite means

Condensed from the provider, a deploy runs exactly this:

```typescript
const vite = await loadVite(rootDir); // your project's own vite install
const builder = await vite.createBuilder({
  root: rootDir,
  define: getDefine(env), // VITE_-prefixed env, see Environment below
  plugins: [cloudflare(pluginOptions), outputPlugin.plugin],
}, null);
await builder.buildApp();
```

A framework is supported by this resource if — and only if — a `vite build` of your `vite.config.ts` builds the whole app. Frameworks that drive their own build never enter this path; they have dedicated resources instead — see [Astro](astro.md), [Nuxt](nuxt.md), [SvelteKit](sveltekit.md), and [Waku](waku.md). For any other build command that produces a directory of files, use [StaticSite](static-site.md).

## Remove @cloudflare/vite-plugin

## Props

`ViteProps` is `WorkerProps` minus the things the build now owns (`main`, the directory-shaped `assets`), plus Vite-specific options:

```typescript
const site = yield* Cloudflare.Website.Vite("Website", {
  rootDir: "./web",
  assets: { runWorkerFirst: true },
});
```

- **`rootDir`** — Vite’s project root, default `process.cwd()`.
- **`memo`** — narrows which files are hashed to decide whether a rebuild is needed (see [Rebuilds and memo](#rebuilds-and-memo)).
- **`viteEnvironments`** — for frameworks that build more than one server environment (see [Multiple server environments](#multiple-server-environments-rsc)).
- **`assets`** — a flat `AssetsConfig` for routing behavior: `runWorkerFirst`, `htmlHandling`, `notFoundHandling`, etc. The built asset directory is supplied by the build, so unlike a plain Worker there is no `directory` to configure.
- Everything else is inherited from the Worker — `domain`, `env`, `compatibility`, `crons`, and so on. See [Workers](../compute/workers.md).

SSR server bundles routinely use Node APIs at runtime — the `nodejs_compat` compatibility flag covers that, and it is enabled by default for every Worker, so there is nothing to pass.

Like `Worker`, the resource also has a class form — only needed when *other* resources bind to the site (the class gives it a named type they can reference):

```typescript
export class Website extends Cloudflare.Website.Vite<Website>()("Website", {}) {}
```

For everything else — including deriving `env` types — a plain `const` is enough; see [Runtime bindings](#runtime-bindings) below.

## Environment

`env` feeds your app through two distinct channels: build-time inlining into the client bundle, and runtime Worker bindings for server code.

### Build-time inlining

Only `VITE_` -prefixed keys are inlined into the client bundle as `import.meta.env.<KEY>`, matching `vite build` ’s default `envPrefix` semantics:

```typescript
const web = yield* Cloudflare.Website.Vite("Website", {
  env: {
    VITE_API_URL: backend.url.as<string>(),
  },
});
```

`Output` values like `backend.url` resolve at deploy time before the build runs, so the client reads a concrete `import.meta.env.VITE_API_URL`. `Redacted` values are unwrapped when `VITE_` -prefixed — the prefix means you are opting the value into the public bundle, so don’t prefix secrets with `VITE_`.

### The site’s own URL

`backend.url` works for *another* Worker’s URL, but a site can’t reference its own `url` Output. `Worker.URL` closes the loop: Alchemy resolves the URL the Worker will be served at — its first custom domain, otherwise its `workers.dev` URL — before the build and injects it:

```typescript
const web = yield* Cloudflare.Website.Vite("Website", {
  env: {
    VITE_PUBLIC_URL: Cloudflare.Worker.URL,
  },
});
```

The client bundle reads a concrete `import.meta.env.VITE_PUBLIC_URL`, and server code sees the same value on `env.VITE_PUBLIC_URL` (typed `string` by `InferEnv`). Under `alchemy dev` it resolves to the local dev server’s URL.

### Runtime bindings

Non- `VITE_` values become native Worker bindings, available to SSR code and typed via `Cloudflare.InferEnv`:

```typescript
export const Website = Cloudflare.Website.Vite("Website", {
  env: {
    BUCKET: Bucket,
    BACKEND: Backend,
  },
});

export type WebsiteEnv = Cloudflare.InferEnv<typeof Website>;
```

See [TanStack Start](tanstack-start.md) for the full pattern of consuming these bindings from server routes.

## Multiple server environments (RSC)

`viteEnvironments` defaults to `{ entry: "ssr", children: [] }`. Frameworks that emit several server environments — React Server Components split into `rsc` and `ssr` — declare which environment produces the Worker entry chunk and which are bundled alongside it:

```typescript
const app = yield* Cloudflare.Website.Vite("ReactRouterRSC", {
  viteEnvironments: { entry: "rsc", children: ["ssr"] },
});
```

The `entry` environment becomes the deployed Worker entry, `children` chunks are bundled alongside it, and the `client` environment is always deployed as static assets. See [React Router](react-router.md) for the worked example.

## Rebuilds and memo

By default, every non-gitignored file under `rootDir` plus the nearest lockfile is content-hashed; if nothing changed, both the build and the deploy are skipped entirely. The hash is path-insensitive — relocating `rootDir` with identical sources is a no-op deploy. Narrow the scope with `memo` when the project contains large directories that don’t affect the build output:

```typescript
const site = yield* Cloudflare.Website.Vite("Docs", {
  memo: {
    include: ["src/**", "index.html", "package.json"],
  },
});
```

## Local dev

```sh
bun alchemy dev
```

`alchemy dev` boots Vite’s own dev server programmatically with the Cloudflare plugin attached — you get HMR on your application code while bindings point at the **real** cloud resources, so there is no emulation fidelity gap. By default the dev server picks an available port; set `dev.port` to keep the local URL stable across runs:

```typescript
const web = yield* Cloudflare.Website.Vite("Website", {
  dev: { port: 3000 },
});
```

## Frameworks

Each supported framework has its own landing page building on this resource:

- [React SPA](vite-spa.md) — single-page apps, from a bare `index.html` to the `create-vite` template.
- [TanStack Start](tanstack-start.md) — full-stack React and Solid with server routes and typed bindings.
- [React Router](react-router.md) — including React Server Components via `viteEnvironments`.
- [Vue](vue.md) — Vue 3 SPAs.
- [SolidStart](solidstart.md) — SolidStart and hand-rolled SolidJS SSR.
- [Octane](octane.md#octane-spas-use-vite-instead) — client-only OctaneJS SPAs (the `octane()` compiler plugin composes with the injected Cloudflare plugin).

[Astro](astro.md), [Nuxt](nuxt.md), [SvelteKit](sveltekit.md), [Waku](waku.md), and fullstack [Octane](octane.md) drive their own builds, so they have dedicated resources (`Website.Astro`, `Website.Nuxt`, `Website.SvelteKit`, `Website.Waku`, `Website.Octane`) instead of this one.
