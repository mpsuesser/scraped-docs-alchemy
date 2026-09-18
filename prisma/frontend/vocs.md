---
url: https://alchemy.run/prisma/frontend/vocs
title: "Vocs"
description: "Deploy Vocs documentation to Prisma Compute with Prisma.Website.Vocs — static assets and Waku RSC on Bun, with Vocs dev locally."
access_date: 2026-09-18T03:55:07.187Z
current_date: 2026-09-18T03:55:07.187Z
---

`Prisma.Website.Vocs` builds a [Vocs](https://vocs.dev/) documentation site and runs it on **Bun in Prisma Compute**. The shared Node target serves client files and prerendered HTML first, then falls through to Vocs’ Waku RSC handler. The result is uploaded as `tar.gz`, not a Docker image. Extensionless pages such as `/about` retain their normal URLs.

## Install

Install the build-time integration; the resource loads `/vocs/node` from your project. Keep `vocs` and its Waku peer dependencies installed.

```sh
bun add -d @alchemy.run/frontend-frameworks @vercel/nft
```

## Configure Vocs

Your `vocs.config.*` loads natively. No adapter is needed:

```typescript
import { defineConfig } from "vocs/config";

export default defineConfig({
  title: "Docs",
  sidebar: [
    { text: "Home", link: "/" },
    { text: "Guide", link: "/guide" },
  ],
});
```

Vocs controls its output directory through `vocs.config.*`; the default is `dist`. If you change it to `build`, also ignore that directory so generated files stay out of the default build-input hash:

```text
build/
```

## Declare the Website

```typescript
import * as Prisma from "alchemy/Prisma";

export const Website = Prisma.Website.Vocs("Website", {
  rootDir: "./docs",
});
```

Omit `rootDir` for a project at `.`. Omit `project` for a database-less Prisma project created on live deploy, or pass an existing project.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyVocsSite",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Website;
    return { url: site.url };
  }),
);
```

`site.url` is the Compute endpoint on deploy and the local Vocs URL in dev.

## Add environment variables

```typescript
export const Website = Prisma.Website.Vocs("Website", {
  rootDir: "./docs",
  env: { DOCS_TITLE: "Hello from Alchemy!" },
});
```

`env` is applied before build and dev, and passed to Compute. These are process environment variables, not Worker bindings.

## Read the environment

```typescript
const title = process.env.DOCS_TITLE ?? "Docs";
```

Prerendered pages capture these values at build time; dynamic pages read the runtime environment. Vocs defaults to dynamic rendering, so its Waku RSC handler stays in the artifact. Deploying only a static HTML shell would drop that handler and leave dynamic pages empty.

## Local development

`bun alchemy dev` runs Vocs’ own dev server with HMR and creates no Prisma resources for the Website. Opt into live deployment during dev:

```typescript
export const Website = Prisma.Website.Vocs("Website", {
  rootDir: "./docs",
}).pipe(Alchemy.remote());
```

## Custom domain

```typescript
export const Website = Prisma.Website.Vocs("Website", {
  rootDir: "./docs",
  domain: "docs.example.com",
});
```

`domain` creates `Prisma.CustomDomain` and changes `site.url` to the HTTPS hostname. The app must be on the project’s current default branch. Configure returned DNS records yourself and verify status before cutover; see [Custom domains](websites.md#custom-domains).

## Where next

- [Vocs API](https://alchemy.run/providers/prisma/website/vocs).
- [Vocs example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-website-vocs).
- [Websites](websites.md) and [Compute apps](../compute/apps.md).
