---
url: https://alchemy.run/railway/compute/environments
title: "Environments"
description: "Extra deploy environments under a Project. Production is created with the Project — do not duplicate it."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A [`Railway.Environment`](https://alchemy.run/providers/railway/reference/project#environment) is an extra
deploy environment under a [Project](projects.md)
(staging, preview, …). The production environment is created with
the Project — do **not** recreate it as an Environment resource.

## Create an extra environment

Alchemy generates a unique name unless you pass one. Production is
already on the Project as `environmentId`.

```typescript
const site = yield* Railway.Project("Site");
const staging = yield* Railway.Environment("Staging", {
  project: site,
});
```

:::note
Prefer omitting `name` in tests and CI so names stay unique and
reclaimable.
:::

## A stable name

Pass `name` when you need a stable environment name (`staging`).
Changing it later updates the environment in place via
`environmentRename`.

```typescript
const staging = yield* Railway.Environment("Staging", {
  project: site,
  name: "staging",
});
```

## Fork from production

`sourceEnvironmentId` copies services, volumes, configuration, and
variables from another environment. Create-only.

```typescript
const staging = yield* Railway.Environment("Staging", {
  project: site,
  sourceEnvironmentId: site.environmentId,
});
```

## Target an environment from a child

Services, Volumes, Postgres, Redis, Variables, and Buckets default
to the Project's primary environment. Pass `environment` to put them
in staging.

```typescript
const api = yield* Railway.Service("StagingApi", {
  project: site,
  environment: staging,
  image: "hashicorp/http-echo",
  port: 5678,
});
```

:::caution[Changing `project` replaces the Environment]
The Environment is created in the new Project. The old Environment
is deleted.
:::

## Module-scope declarations

Declare the Project once. Pass it into every child.

```typescript
// src/project.ts
import * as Railway from "@alchemy.run/railway";

export const Site = Railway.Project("Site");
export const Staging = Railway.Environment("Staging", {
  project: Site,
});
```

Resource-valued props accept the resource or an Effect producing it.

## Where next

[Projects](projects.md) own the production environment.
[Services](services.md) take `environment`.
The [`Environment` reference](https://alchemy.run/providers/railway/reference/project#environment) lists
every prop.
