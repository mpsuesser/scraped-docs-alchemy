---
url: https://alchemy.run/modal/workspaces-and-tokens
title: "Workspaces and tokens"
description: "How a Modal workspace holds everything, how environments keep stages apart, how profiles choose the workspace, and the service users your runtimes act as."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything you create on Modal belongs to a workspace, and every call to Modal carries a token for one.

## A workspace holds everything

A **workspace** holds your apps, images, Volumes, Dicts, Queues and Secrets, the people who may use them, and the bill. A personal account is a workspace of one. A team shares one, and its members get a role in each environment.

**Environments** divide the workspace. Every workspace starts with `main`. Names are unique per environment, so a `Weights` Volume in `prod` and one in `dev` are two Volumes.

acmeworkspace

MyEnv-devenvironment

Coderapp

WeightsVolume

MyEnv-prodenvironmentApi service user: contributor

Coderapp

WeightsVolume

Two stages, two environments, with the same names inside each. The Api Worker's service user is a contributor in the prod environment and has no role in dev.

## Environments keep stages apart

Declare one environment and put everything in it. Alchemy names it from your app, stage and id, so `alchemy deploy --stage prod` creates a second environment beside the first:

```typescript
import * as Modal from "@alchemy.run/modal";

export const MyEnv = Modal.Environment("MyEnv", {});

export const Coder = Modal.App("Coder", { environment: MyEnv });

export const Weights = Modal.Volume("Weights", { environment: MyEnv });
```

An environment can cap what its stage spends:

```typescript
export const MyEnv = Modal.Environment("MyEnv", {
  maxConcurrentTasks: 50,
  maxConcurrentGpus: 4,
  budget: 200,
});
```

Anything that doesn’t name an environment lands in the workspace’s default one.

Alchemy reaches a workspace through a [profile](../environments/profiles.md). Add Modal to your `default` profile:

```sh
pnpm alchemy profile edit --add Modal
```

Pick **Browser login**. Alchemy opens modal.com, you approve, and Modal creates a token for the workspace you choose, the same flow as `modal token new`. To paste an existing token instead, pick **Token**. See [Setup](setup.md).

A token is a token id (`ak-…`) and a secret (`as-…`). It acts on everything in its workspace, and doesn’t expire.

## Give production its own workspace

The [stage](../environments/stages.md) decides **what** you deploy: its own environment and the names inside it. The profile decides **where**: which workspace’s token Alchemy uses.

```sh
pnpm alchemy profile create prod
pnpm alchemy profile edit --profile prod --add Modal
pnpm alchemy deploy --stage prod --profile prod
pnpm alchemy deploy --stage pr-42 --profile default
```

Many stages can share a workspace, each in its own environment. Give production a workspace of its own when you want its members and bill kept apart too.

## Each runtime gets a service user

Your runtimes never get your token. When a Worker or Lambda binds a Modal object, deploy creates a **service user** for it and grants it a role in the bound object’s environment: `viewer` to read, `contributor` to write or create.

```typescript
const sandboxes = yield* Modal.CreateSandbox(Warm);
```

This Worker’s service user is a `contributor` in `Warm` ’s environment, and nothing in any other. Keep workloads that shouldn’t touch each other in separate environments.

A Modal Function or Server needs no service user: it calls Modal as its own container, so no token is stored in it.

Declare a service user yourself for anything outside the Stack, like a CI job:

```typescript
export const Deployer = Modal.ServiceUser("Deployer", {
  environments: [{ environment: MyEnv, role: "contributor" }],
});
```

Its `tokenId` and `tokenSecret` are a token pair like yours, that only works in the environments you list.

Service users need Modal’s Team or Enterprise plan. See [Access control](security/access-control.md) for the Starter plan, and for environments only some members may change.

## Deploy from CI with a token

Set the same variables the `modal` CLI reads:

```sh
MODAL_TOKEN_ID=ak-...
MODAL_TOKEN_SECRET=as-...
```

Use a workspace of its own for tests. See [CI](../environments/ci.md).

To give your code an API key or a password, see [Secrets](security/secrets.md).
