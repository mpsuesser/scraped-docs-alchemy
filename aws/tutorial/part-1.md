---
url: https://alchemy.run/aws/tutorial/part-1
title: "Part 1: Your First Stack"
description: "Install Alchemy, create a Stack with an AWS S3 Bucket, and deploy it."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
---

In this first part you’ll install Alchemy and Effect, create a Stack with an AWS S3 Bucket, and deploy it — all in under five minutes.

## Prerequisites

- [Bun](https://bun.sh/) (recommended) or Node.js 22+
- An AWS account and an IAM identity with permission to create the resources you plan to deploy

## Create a project

Start with an empty directory and initialize a `package.json`:

```sh
mkdir my-app && cd my-app && bun init -y
```

## Install dependencies

Install `alchemy@latest` and `effect@rc`:

```sh
bun add "alchemy@latest" "effect@rc" "@effect/platform-bun@rc" "@effect/platform-node@rc"
```

## Create the Stack

Every Alchemy program starts with a `Stack` — a collection of Resources managed by Providers with state tracked between deploys.

Create an `alchemy.run.ts` file:

```typescript
import * as Alchemy from "alchemy";
import * as AWS from "alchemy/AWS";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: AWS.providers(),
    state: AWS.state(),
  },
  Effect.gen(function* () {
    // we'll add resources here next
  }),
);
```

`Alchemy.Stack` is the root of every app. The first argument (`"MyApp"`) doubles as a logical id and as the prefix Alchemy uses when AWS asks for physical resource names.

## How credentials resolve

`providers: AWS.providers()` registers every AWS resource and IAM policy binding that ships with Alchemy, and resolves credentials from your Alchemy [profile](../../environments/profiles.md). Connect SSO or stored access keys with `alchemy profile edit --add AWS`. See [Setup](../setup.md) for the details of each method.

## Where state lives

`state: AWS.state()` stores deploy state in an account-regional S3 bucket (`alchemy-state-{accountId}-{region}-an`), created lazily on the first deploy. Because the state lives in your AWS account rather than on disk, the same configuration works locally and in CI with no extra setup.

## Add a Resource

Resources represent cloud infrastructure — buckets, tables, queues, functions, and so on. Each resource is `yield*` -ed inside the Stack’s Effect generator.

Add an S3 Bucket:

```typescript
import * as Alchemy from "alchemy";
import * as AWS from "alchemy/AWS";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: AWS.providers(),
    state: AWS.state(),
  },
  Effect.gen(function* () {
    const bucket = yield* AWS.S3.Bucket("Bucket");
  }),
);
```

`yield* AWS.S3.Bucket("Bucket")` registers the resource with the Stack under the logical id `Bucket` and returns a typed `Bucket` handle. We didn’t pass a `bucketName`, so Alchemy generates a unique physical name from the app, stage, and logical id.

## Return Stack outputs

Stack outputs let you see important values after a deploy. Return an object from the generator to expose them:

```typescript
Effect.gen(function* () {
  const bucket = yield* AWS.S3.Bucket("Bucket");

  return {
    bucketName: bucket.bucketName,
  };
}),
```

`bucket.bucketName` is an Output — a typed reference to an attribute the cloud will produce. It resolves during deploy and prints with the rest of the stack outputs.

## Deploy

Run `alchemy deploy` to create the Bucket on AWS:

```sh
bun alchemy deploy
```

Connect AWS first with `alchemy profile edit --add AWS` and pick a local authentication method: **SSO** (recommended) or **stored access keys**. The choice is saved to your **`default`** [profile](../../environments/profiles.md); the region and account come from whichever method you pick, and a deploy with nothing configured fails with that exact command to run. CI instead reads AWS environment credentials directly. See [Setup](../setup.md) for the full walkthrough.

```
Plan: 1 to create
+ Bucket (AWS.S3.Bucket)

Proceed?
◉ Yes ○ No
✓ Bucket (AWS.S3.Bucket) created
{
  bucketName: "myapp-bucket-a1b2c3d4e5",
}
```

Alchemy shows a plan, asks for confirmation, creates the resource, and prints the stack outputs. Your bucket is live on AWS.

## Verify it worked

Your newly created bucket will be listed in the S3 section of the AWS Console (or via `aws s3 ls` if you have the AWS CLI installed).

Run `alchemy deploy` again. Because nothing changed, the bucket shows as a no-op:

```
Plan: no changes

{
  bucketName: "myapp-bucket-a1b2c3d4e5",
}
```

This is the core loop — declare resources in code, deploy, and Alchemy figures out what changed.

## Destroy (optional)

The inverse of `deploy` is `destroy` — it plans against an empty desired state and deletes everything the stack created:

```sh
bun alchemy destroy
```

Try it if you like, then run `bun alchemy deploy` again to bring the bucket back — we’ll build on it in Part 2.

## Recap

You now have:

- An `alchemy.run.ts` with a Stack and an AWS S3 Bucket
- A live bucket deployed to your AWS account
- Stack outputs showing the bucket name

In [Part 2](part-2.md), you’ll add a Lambda Function with a public URL that reads and writes objects in this bucket.
