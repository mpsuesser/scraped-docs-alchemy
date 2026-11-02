---
url: https://alchemy.run/railway/compute/projects
title: "Projects"
description: "A workspace-scoped namespace containing Services, Environments, Volumes, Variables, and Buckets."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A [`Railway.Project`](https://alchemy.run/providers/railway/reference/project#project) is a workspace-scoped
namespace. It owns environments and services. Names are unique per
workspace.

## Create a Project

Alchemy generates a unique name unless you pass one.

```typescript
const site = yield* Railway.Project("Site");
```

`url` is `https://railway.com/project/{projectId}`. A production
environment is created with the project — do not recreate it as an
[Environment](environments.md). `environmentId` is that
primary environment.

Nothing answers on the public internet until a
[Service](services.md) is deployed. The Service gets its
own `*.up.railway.app` hostname.

:::note
Prefer omitting `name` in tests and CI so names stay unique and
reclaimable.
:::

## A stable name

Pass `name` when you need a stable project name.

```typescript
const site = yield* Railway.Project("Site", {
  name: "my-site",
});
```

Changing `name` later updates the project in place.

## Description

`description` is optional and updates in place.

```typescript
const site = yield* Railway.Project("Site", {
  description: "production web app",
});
```

## Workspace

Workspace defaults to the current token
(`me.workspace ?? me.workspaces[0]`). Pass `workspaceId` to pin it.

```diff lang="typescript"
const site = yield* Railway.Project("Site", {
  name: "my-site",
+  workspaceId: "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
});
```

:::caution[Changing `workspaceId` replaces the Project]
The Project is created in the new workspace. The old Project is
deleted.
:::

## Default environment name

`defaultEnvironmentName` is create-only. Railway's default is
`production`. Extra environments (staging) are a separate
[Environment](environments.md) resource.

```typescript
const site = yield* Railway.Project("Site", {
  defaultEnvironmentName: "production",
});
```

## Module-scope declarations

Declare the Project once. Pass it into every child.

```typescript
// src/project.ts
import * as Railway from "@alchemy.run/railway";

export const Site = Railway.Project("Site");
```

Resource-valued props accept the resource or an Effect producing
it. Do not unwrap it just to pass it along.

## Destroy

`alchemy destroy` deletes children, then the Project.

## Where next

The [tutorial](../tutorial/part-1.md) deploys a Project from an
empty directory. [Services](services.md) run in the
Project. [Environments](environments.md) add staging.
The [`Project` reference](https://alchemy.run/providers/railway/reference/project#project) lists every
prop.
[Example: railway-project](https://github.com/alchemy-run/alchemy/tree/main/examples/railway-project)
is a Project plus an image Service.
