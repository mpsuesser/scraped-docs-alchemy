---
url: https://alchemy.run/e2b/setup
title: "Setup"
description: "Install @alchemy.run/e2b, sign in to E2B with a profile, and deploy from CI."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

You need:

- [Node.js](https://nodejs.org/) 24.11.1+ or [Bun](https://bun.sh/)
- An E2B account, from [e2b.dev](https://e2b.dev/)

## Add the provider

Install `@alchemy.run/e2b`:

```sh
pnpm add @alchemy.run/e2b@latest
```

Then add `E2B.providers()` next to your cloud’s providers:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as E2B from "@alchemy.run/e2b";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), E2B.providers()),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    return {};
  }),
);
```

## Sign in

```sh
pnpm alchemy profile edit --add E2B
```

Pick **Browser login** and sign in to E2B. Alchemy deploys to your account’s default team, and the login refreshes itself. To paste an access token or an API key instead, pick **Access token or API key**.

E2B can’t narrow an API key, so each one reaches everything in its team. Use a separate team for tests. See [Team & API keys](api-keys.md).

## Deploy from CI

Set `E2B_ACCESS_TOKEN`, and Alchemy uses it instead of your profile:

```sh
E2B_ACCESS_TOKEN=sk_e2b_... pnpm alchemy deploy
```

`E2B_API_KEY` is enough to deploy resources, but bindings and `E2B.ApiKey` create keys, which takes the access token. See [Team & API keys](api-keys.md#deploy-from-ci-with-a-token).

Next, build the [tutorial](tutorial/part-1.md).
