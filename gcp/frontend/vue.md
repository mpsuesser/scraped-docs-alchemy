---
url: https://alchemy.run/gcp/frontend/vue
title: "Vue"
description: "Deploy a Vue single-page app to Cloud Run with GCP.Website.Vite — scaffold with create-vue, deep links answered by index.html."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

A Vue app is a Vite app, so it deploys with [`GCP.Website.Vite`](vite.md). Vue Router’s history mode works out of the box: unknown paths answer with `index.html`.

## Install

```sh
pnpm add -D @alchemy.run/frontend-frameworks@latest
```

Docker has to be running when you deploy.

## Scaffold the app

```sh
pnpm create vite web --template vue-ts
cd web && pnpm install && cd ..
```

Keep the generated `vite.config.ts` as it is.

## Deploy the site

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export const Website = GCP.Website.Vite("Website", { rootDir: "./web" });

export default Alchemy.Stack(
  "MyVueApp",
  { providers: GCP.providers(), state: GCP.state() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

```sh
bun alchemy deploy
```

When you add [Vue Router](https://router.vuejs.org/) in history mode, deep links work as they are: the server answers unknown paths with `index.html`.

## Local development

`alchemy dev` runs Vite’s dev server with hot reload.

## Where next

- [Vite](vite.md) — env values, base paths, and 404 pages.
- [Nuxt](nuxt.md) — when you need server rendering.
