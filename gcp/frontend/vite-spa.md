---
url: https://alchemy.run/gcp/frontend/vite-spa
title: "React SPA"
description: "Deploy a React single-page app to Cloud Run with GCP.Website.Vite — scaffold with Vite, deep links answered by index.html, Vite's dev server under alchemy dev."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

This walks a plain React single-page app onto Cloud Run with [`GCP.Website.Vite`](vite.md). Client-side routes work out of the box: unknown paths answer with `index.html`.

## Install

```sh
pnpm add -D @alchemy.run/frontend-frameworks@latest
```

Docker has to be running when you deploy.

## Scaffold the app

Start from Vite’s React + TypeScript template:

```sh
pnpm create vite web --template react-ts
cd web && pnpm install && cd ..
```

The app lands in `./web/` with its own `vite.config.ts`. Keep it as it is. You don’t need an adapter.

## Deploy the site

Point `rootDir` at the app:

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export const Website = GCP.Website.Vite("Website", { rootDir: "./web" });

export default Alchemy.Stack(
  "MyReactApp",
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

## Add client-side routes

Install React Router in the app:

```sh
pnpm add react-router-dom
```

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import App from "./App.tsx";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<App />} />
        <Route path="/about" element={<h1>About</h1>} />
      </Routes>
    </BrowserRouter>
  </StrictMode>,
);
```

Deploy again and open `/about` directly. The server answers with `index.html` and the router renders the page.

## Call an API

Pass a Cloud Run service’s URL to the build:

```typescript
Effect.gen(function* () {
  const api = yield* Api;
  const site = yield* Website;
  const site = yield* GCP.Website.Vite("Website", {
    rootDir: "./web",
    env: { VITE_API_URL: api.uri },
  });
  return { url: site.url };
}),
```

Read it in the app with `import.meta.env.VITE_API_URL`. The API needs CORS headers for the site’s origin.

## Local development

`alchemy dev` runs Vite’s dev server with hot reload.

## Where next

- [Vite](vite.md) — every option the resource takes.
- [Cloud Run](../compute/cloud-run.md) — build the API the app calls.
