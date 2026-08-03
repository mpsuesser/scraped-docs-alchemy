---
url: https://alchemy.run/getting-started
title: "Getting started"
description: "Install Alchemy and create your first Stack in under two minutes."
access_date: 2026-08-03T19:08:21.153Z
current_date: 2026-08-03T19:08:21.153Z
---

## Prerequisites

- [Bun](https://bun.sh/) (recommended) or Node.js 22+
- A [Cloudflare](https://dash.cloudflare.com/sign-up) account

## Create a project

```sh
mkdir my-app && cd my-app && bun init -y
```

## Install

```sh
bun add "alchemy@next" "effect@>=4.0.0-beta.102 || >=4.0.0" "@effect/platform-bun@>=4.0.0-beta.102 || >=4.0.0" "@effect/platform-node@>=4.0.0-beta.102 || >=4.0.0"
```

## Create your Stack

Every Alchemy program starts with a Stack — create `alchemy.run.ts`:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "alchemy/Cloudflare";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Cloudflare.providers(),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const bucket = yield* Cloudflare.R2.Bucket("Bucket");

    return {
      bucketName: bucket.bucketName,
    };
  }),
);
```

## Deploy

Run `alchemy deploy` to create the Bucket on Cloudflare:

```sh
bun alchemy deploy
```

The first time you deploy, Alchemy walks each provider in your stack (here, Cloudflare) through an interactive login and saves the credentials to your **`default`** [profile](environments/profiles.md) at `~/.alchemy/profiles.json`. For Cloudflare you can sign in with OAuth in the browser or paste an API token — no environment variables required.

Once you’re authenticated, Alchemy shows a plan, asks for confirmation, and provisions the resource:

```
Plan: 1 to create
+ Bucket (Cloudflare.R2.Bucket)

Proceed?
◉ Yes ○ No
✓ Bucket (Cloudflare.R2.Bucket) created
{
  bucketName: "myapp-bucket-a1b2c3d4e5",
}
```

That’s it — you have a live R2 Bucket on Cloudflare.

Full command reference: [CLI](cli.md).
