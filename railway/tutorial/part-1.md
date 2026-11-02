---
url: https://alchemy.run/railway/tutorial/part-1
title: "Part 1: Your First Project"
description: "Install Alchemy, create a Stack with a Railway Project, and deploy it."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

Install Alchemy and Effect, create a Stack with a Railway Project, and deploy it. A Project is the parent for Services, Environments, Variables, and disks.

## Prerequisites

- [Bun](https://bun.sh/) or Node.js 22+
- A Railway workspace and an account API token — see [Setup](../setup.md) if you haven’t created those yet

## Create a project

Start with an empty directory and initialize a `package.json`:

```sh
mkdir my-app && cd my-app && pnpm init
```

## Install dependencies

Install `alchemy@latest`, `@alchemy.run/railway`, and `effect`:

```sh
pnpm add alchemy@latest @alchemy.run/railway@latest effect @effect/platform-bun @effect/platform-node
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

Railway has no state backend of its own, so we’ll keep state on disk with `Alchemy.localState()` — it writes to `.alchemy/` next to your code, no setup required:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.empty,
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    // we'll add resources here next
  }),
);
```

## Add a Project

Resources represent cloud infrastructure. Each resource is `yield*` -ed inside the Stack’s Effect generator.

Let’s declare a Railway Project and observe the type error:

```typescript
import * as Alchemy from "alchemy";
import * as Railway from "@alchemy.run/railway";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.empty,Error ts(2322)  ― Type 'Layer<never, never, never>' is not assignable to type 'Layer<NoInfer<Providers>, never, StackServices>'.
  Type 'Providers' is not assignable to type 'never'.
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const site = yield* Railway.Project("Site");
  }),
);
```

Omit `name` and Alchemy generates a unique, DNS-safe Project name (max 32 characters, leading letter). The workspace defaults to whatever workspace your token belongs to. A production environment is created with the Project.

TypeScript is telling us that `Layer.empty` doesn’t provide `Railway.Providers` — the layer required by `Project`.

## Fix the Providers

Replace `Layer.empty` with `Railway.providers()` to resolve the type error:

```typescript
import * as Alchemy from "alchemy";
import * as Railway from "@alchemy.run/railway";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Railway.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const site = yield* Railway.Project("Site");
  }),
);
```

Now the program type-checks. The providers layer tells Alchemy how to talk to the Railway GraphQL API, and the type system ensures you never forget to wire it up.

## Return Stack outputs

Stack outputs let you see important values after a deploy. Return an object from the generator to expose the Project’s name, id, and dashboard URL:

```typescript
Effect.gen(function* () {
  const site = yield* Railway.Project("Site");

  return {
    name: site.name,
    projectId: site.projectId,
    url: site.url,
  };
}),
```

`url` is `https://railway.com/project/{projectId}`. Nothing is listening on the public internet yet — that’s [Part 2](part-2.md).

## Deploy

Run `alchemy deploy` to create the Project:

```sh
pnpm alchemy deploy
```

The first time you deploy, Alchemy prompts for Railway credentials — paste the account token you generated in [Setup](../setup.md). The token is verified and saved to your **`default`** [profile](../../environments/profiles.md), so you won’t be asked again.

```
Plan: 1 to create

+ Site (Railway.Project)

Proceed?
◉ Yes ○ No
✓ Site (Railway.Project) created
{
  name: "myapp-site-dev-a1b2c3d4",
  projectId: "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  url: "https://railway.com/project/aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
}
```

Alchemy shows a plan, asks for confirmation, creates the Project, and prints the stack outputs.

## Verify it worked

The Project is listed in your workspace in the [Railway dashboard](https://railway.com/dashboard). Run `alchemy deploy` again. Because nothing changed, the Project shows as a no-op:

```
Plan: no changes

{
  name: "myapp-site-dev-a1b2c3d4",
  projectId: "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  url: "https://railway.com/project/aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
}
```

This is the core loop — declare resources in code, deploy, and Alchemy figures out what changed.

## Recap

You now have:

- An `alchemy.run.ts` with a Stack and a Railway Project
- A live Project in your workspace with a generated unique name
- A production environment created with the Project
- Stack outputs showing `name`, `projectId`, and the dashboard URL

In [Part 2](part-2.md), you’ll deploy code to this Project — a `Railway.Service` that bundles an Effect HTTP handler into a container.
