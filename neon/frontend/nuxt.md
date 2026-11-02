---
url: https://alchemy.run/neon/frontend/nuxt
title: "Nuxt"
description: "Package Nuxt SSR and public assets for Neon Functions, with native Nuxt development."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

`Neon.Website.Nuxt` builds Nuxt with Nitro's Node target, then replaces the listening
wrapper with a Node 24 Fetch entry for Neon Functions. SSR and public assets ship
in the Function ZIP, not a container or S3 website.

## Prepare the app

Use an existing Nuxt app or the
[runnable Nuxt example](https://github.com/alchemy-run/alchemy/tree/main/examples/neon-website-nuxt).
Configure [deployment credentials](../setup.md), then install in the app directory:

Install [Alchemy and its matching Effect dependencies](../../getting-started.md) first;
use Node 24 for this build toolchain.

```sh
pnpm add -D @alchemy.run/frontend-frameworks @vercel/nft
```

Keep modules and application settings in `nuxt.config.*`. Do not override
`nitro.preset`: the target owns it. The `nuxt` resource prop accepts serializable
config overrides; keep functions and plugin instances in the config file.

## Define the stack

```typescript
// alchemy.run.ts
import * as Alchemy from "alchemy";
import * as Neon from "@alchemy.run/neon";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "NeonNuxt",
  { providers: Neon.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const site = yield* Neon.Website.Nuxt("Web", {
      rootDir: ".",
    });
    return { url: site.url };
  }),
);
```

Nitro handles server routes; built public files are served from the same Function.
Use Nuxt's `runtimeConfig` for application configuration and its normal
`NUXT_*` environment mapping. Values in `runtimeConfig.public` are browser-visible.
The Website's `env` is also available during build and native development.

## Develop and attempt deployment

```sh
pnpm exec alchemy dev
pnpm exec alchemy deploy
pnpm exec alchemy destroy
```

Run separately, destroying when finished. Dev uses Nuxt's native server and HMR,
returns a local URL, and creates no implicit Project or Function. Append
`.pipe(Alchemy.remote())` to the Website Effect to opt into live deployment during dev.

Live deployment creates an Ohio Project when scope is omitted. Set either
`project` or `branch` for an existing backend; choose Ohio when adding storage.
Read [scope and credentials](vite.md#scope-and-credentials) before
passing backend tokens to application code.

:::caution[Production verification remains open]
The verification run encountered a Nuxt prerender failure in its temporary
dependency setup. Native dev success does not establish production packaging or
live acceptance. Verify your production artifact and actual Function responses;
Neon probes have served old code/env after successful deployment metadata updates.
:::

No CDN, durable Nitro cache, or image-optimization service is provisioned. Domain
registration does not validate TLS. See the
[shared deployment limits](vite.md#environment-and-deployment-limits).

## Reference

- [Nuxt API](https://alchemy.run/providers/neon/website#nuxt)
- [Runnable Nuxt example](https://github.com/alchemy-run/alchemy/tree/main/examples/neon-website-nuxt)
