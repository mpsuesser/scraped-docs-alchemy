---
url: https://alchemy.run/getting-started
title: "Getting started"
description: "Install Alchemy and create your first Stack in under two minutes."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
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
bun add "alchemy@latest" "effect@rc" "@effect/platform-bun@rc" "@effect/platform-node@rc"
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

Connect Cloudflare first with `alchemy profile edit --add Cloudflare`. Sign in with OAuth in the browser or paste an API token. The credentials are saved to your **`default`** [profile](environments/profiles.md) at `~/.alchemy/profiles.json`. No environment variables required. Logging in only ever happens through the `profile` command; a deploy with nothing configured fails with the exact command to run.

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
