---
url: https://alchemy.run/gcp/tutorial/part-1
title: "Part 1: Your First Stack"
description: "Install Alchemy, create a Stack with a Cloud Storage Bucket, and deploy it to Google Cloud."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

In this part you install Alchemy, create a Stack with a Cloud Storage Bucket, and deploy it to your Google Cloud project.

## Prerequisites

- [Bun](https://bun.sh/) or Node.js 24.11.1+
- A Google Cloud project and a service-account key that can create what you deploy. See [Setup](../setup.md).

## Create a project

Start with an empty directory and a `package.json`:

```sh
mkdir my-app && cd my-app && pnpm init
```

## Install dependencies

Install Alchemy, the GCP provider, and Effect:

```sh
pnpm add alchemy@latest @alchemy.run/gcp@latest effect @effect/platform-bun @effect/platform-node
```

## Create the Stack

Every Alchemy program starts with a `Stack`. Create `alchemy.run.ts`:

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: GCP.providers(),
    state: GCP.state(),
  },
  Effect.gen(function* () {
    // resources go here next
  }),
);
```

The first argument, `"MyApp"`, names the Stack. Alchemy uses it as a prefix when it generates names for your resources.

`GCP.providers()` registers every GCP resource.

## Where state lives

`state: GCP.state()` keeps deploy state in a Cloud Storage bucket in your project, `alchemy-state-{project}`, created on the first deploy. Because state lives in your project and not on your machine, the same setup works on your laptop and in CI.

## Connect your project

Point Alchemy at your project once:

```sh
bun alchemy profile edit --add GCP
```

Choose **Service account JSON**, pick the key file, and pick a default region. The choice is saved to your `default` [profile](../../environments/profiles.md).

## Add a Bucket

Declare a Cloud Storage Bucket inside the Stack:

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: GCP.providers(),
    state: GCP.state(),
  },
  Effect.gen(function* () {
    const bucket = yield* GCP.Storage.Bucket("Bucket", {});
  }),
);
```

`Bucket` is the logical id. Alchemy generates a globally unique bucket name from the Stack, stage, and logical id.

## Return Stack outputs

Return the values you want to see after a deploy:

```typescript
Effect.gen(function* () {
  const bucket = yield* GCP.Storage.Bucket("Bucket", {});

  return {
    bucketName: bucket.bucketName,
  };
}),
```

`bucket.bucketName` is an Output. It resolves during the deploy and prints with the rest of the outputs.

## Deploy

```sh
pnpm alchemy deploy
```

```
Plan: 1 to create
+ Bucket (GCP.Storage.Bucket)

Proceed?
◉ Yes ○ No
✓ Bucket (GCP.Storage.Bucket) created
{
  bucketName: "myapp-live-sam-bucket-a1b2c3d4",
}
```

Alchemy shows the plan, asks for confirmation, creates the Bucket, and prints the outputs.

## Verify it worked

The Bucket shows up under **Cloud Storage → Buckets** in the console, or with:

```sh
gcloud storage ls
```

Deploy again. Nothing changed, so there is nothing to do:

```
Plan: no changes

{
  bucketName: "myapp-live-sam-bucket-a1b2c3d4",
}
```

## Destroy (optional)

`destroy` deletes everything the Stack created:

```sh
bun alchemy destroy
```

Deploy again afterwards if you try it. Part 2 builds on the Bucket.

## Recap

You now have:

- An `alchemy.run.ts` with a Stack and a Cloud Storage Bucket
- A live Bucket in your Google Cloud project
- Stack outputs showing the Bucket name

In [Part 2](part-2.md), you put an HTTP service on Cloud Run that reads and writes objects in this Bucket.
