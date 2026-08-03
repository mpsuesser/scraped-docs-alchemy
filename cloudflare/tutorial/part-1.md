---
url: https://alchemy.run/cloudflare/tutorial/part-1
title: "Part 1: Your First Stack"
description: "Install Alchemy, create a Stack with a Cloudflare R2 Bucket, and deploy it."
access_date: 2026-08-03T19:08:21.153Z
current_date: 2026-08-03T19:08:21.153Z
---

In this first part you’ll install Alchemy and Effect, create a Stack with a Cloudflare R2 Bucket, and deploy it — all in under five minutes.

## Prerequisites

- [Bun](https://bun.sh/) (recommended) or Node.js 22+
- A [Cloudflare](https://dash.cloudflare.com/sign-up) account

## Create a project

Start with an empty directory and initialize a `package.json`:

```sh
mkdir my-app && cd my-app && bun init -y
```

## Install dependencies

Install `alchemy@next` and `effect@>=4.0.0-beta.102 || >=4.0.0`:

```sh
bun add "alchemy@next" "effect@>=4.0.0-beta.102 || >=4.0.0" "@effect/platform-bun@>=4.0.0-beta.102 || >=4.0.0" "@effect/platform-node@>=4.0.0-beta.102 || >=4.0.0"
```

## Create the Stack

Every Alchemy program starts with a `Stack` — a collection of Resources managed by Providers with state tracked between deploys.

Create an `alchemy.run.ts` file:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {Error ts(2345)  ― Argument of type '{ providers: Layer.Layer<never, never, never>; }' is not assignable to parameter of type 'StackProps<never>'.
  Property 'state' is missing in type '{ providers: Layer.Layer<never, never, never>; }' but required in type 'StackProps<never>'.
    providers: Layer.empty,
  },
  Effect.gen(function* () {
    // we'll add resources here next
  }),
);
```

TypeScript is unhappy: the `state` property is required. Every Stack needs a **state store** so Alchemy can persist resource state between deploys and compute diffs against your infrastructure.

## Configure state

For this tutorial we’ll use `Cloudflare.state()`, which persists state in a Cloudflare-hosted Worker backed by a Durable Object:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "alchemy/Cloudflare";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.empty,
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    // we'll add resources here next
  }),
);
```

The first time you run `alchemy deploy`, `plan`, or `dev`, Alchemy will prompt for permission and bootstrap the state-store Worker into your Cloudflare account. It looks like this:

*Interactive demonstration unavailable in static documentation.*

This is a one-time event — the state-store Worker, its Durable Object, and the Secrets Store entries holding its auth token and encryption key are reused across every stack and stage on this Cloudflare account. See [State Store](../../state-store.md) for the full picture.

## Add a Resource

Resources represent cloud infrastructure — buckets, queues, functions, databases, and so on. Each resource is `yield*` -ed inside the Stack’s Effect generator.

Let’s add a Cloudflare R2 Bucket to our Stack and observe the type error:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "alchemy/Cloudflare";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.empty,Error ts(2322)  ― Type 'Layer<never, never, never>' is not assignable to type 'Layer<NoInfer<Providers>, never, StackServices>'.
  Type 'Providers' is not assignable to type 'never'.
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const bucket = yield* Cloudflare.R2.Bucket("Bucket");
  }),
);
```

TypeScript is telling us that `Layer.empty` doesn’t provide `Cloudflare.Providers` — the layer required by `Bucket`.

## Fix the Providers

Replace `Layer.empty` with `Cloudflare.providers()` to resolve the type error:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "alchemy/Cloudflare";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Cloudflare.providers(),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const bucket = yield* Cloudflare.R2.Bucket("Bucket");
  }),
);
```

Now the program type-checks. The providers layer tells Alchemy how to talk to Cloudflare’s APIs, and the type system ensures you never forget to wire it up.

## Return Stack outputs

Stack outputs let you see important values after a deploy. Return an object from the generator to expose them:

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

The first time you deploy, Alchemy walks each provider in your stack through an interactive login and saves the credentials to your **`default`** [profile](../../environments/profiles.md) at `~/.alchemy/profiles.json`. For Cloudflare you can sign in with OAuth in the browser or paste an API token — no environment variables or `wrangler login` required.

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

Alchemy shows a plan, asks for confirmation, creates the resource, and prints the stack outputs. Your bucket is live on Cloudflare.

## Verify it worked

Your newly created R2 bucket will be listed on the [Cloudflare R2 Object Storage Dashboard](https://dash.cloudflare.com/?to=/:account/r2/overview).

Run `alchemy deploy` again. Because nothing changed, the bucket shows as a no-op:

```
Plan: no changes

{
  bucketName: "myapp-bucket-a1b2c3d4e5",
}
```

This is the core loop — declare resources in code, deploy, and Alchemy figures out what changed.

## Recap

You now have:

- An `alchemy.run.ts` with a Stack and a Cloudflare R2 Bucket
- A live bucket deployed to your Cloudflare account
- Stack outputs showing the bucket name

In [Part 2](part-2.md), you’ll add a Cloudflare Worker that uses this bucket to serve HTTP requests.
