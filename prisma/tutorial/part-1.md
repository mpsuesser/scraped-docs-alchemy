---
url: https://alchemy.run/prisma/tutorial/part-1
title: "Part 1: Your First Project"
description: "Install Alchemy, configure Prisma credentials, and deploy your first Project."
access_date: 2026-09-18T03:55:07.187Z
current_date: 2026-09-18T03:55:07.187Z
---

Build a full-stack application on Prisma in four parts: a Project, an Effect-native Compute API, a Postgres database, and a Vite frontend. The [complete example](https://github.com/alchemy-run/alchemy/tree/main/examples/prisma-tutorial) contains the final application and its deployment test.

## Prerequisites

- [Bun](https://bun.sh/) or Node.js 22+ installed locally.
- A Prisma workspace with access to Compute and Postgres.
- A workspace service token from the [Prisma Console](https://console.prisma.io/).

## Create a directory

```sh
mkdir prisma-tutorial
cd prisma-tutorial
```

Keep all of the following files in this directory.

## Initialize the package

```sh
bun init -y
```

This creates the `package.json` that records your dependencies.

## Install Alchemy

```sh
bun add "alchemy@next" "effect@rc" "@effect/platform-node@rc"
```

Use matching Effect versions for Alchemy and the platform packages.

## Configure credentials

```sh
bun alchemy profile edit --add Prisma
```

Enter your service token when prompted. [Setup](../setup.md) explains profiles and the `PRISMA_SERVICE_TOKEN` environment variable used in CI.

## Ignore local state

Create `.gitignore`:

```text
node_modules/
.alchemy/
dist/
.env
```

Alchemy’s local state contains resource identifiers and potentially sensitive outputs. Keep it out of source control, but retain it between deployments.

## Create the Stack

Create `alchemy.run.ts`:

```typescript
import * as Alchemy from "alchemy";
import * as Prisma from "alchemy/Prisma";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "PrismaTutorial",
  { providers: Prisma.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {}),
);
```

A Stack groups resources. The Prisma providers manage them, and local state records what this Stack owns. For shared deployments, use a [shared state store](../../state-store.md) instead.

## Define a Project

Create `src/Database.ts`:

```typescript
import * as Prisma from "alchemy/Prisma";

export const Project = Prisma.Project("Project", {
  createDatabase: false,
  region: "eu-west-3",
});
```

The Project will contain both Compute services and Postgres. Disable the implicit default database because Part 3 creates an explicit database resource. Alchemy generates a physical name from the Stack, stage, and logical ID.

## Add the Project to the Stack

```typescript
import { Project } from "./src/Database.ts";

  Effect.gen(function* () {}),
  Effect.gen(function* () {
    const project = yield* Project;
    return { projectId: project.projectId };
  }),
```

Yielding the resource registers it in the Stack. Returning its ID makes that value visible in the deployment output.

## Deploy

```sh
bun alchemy deploy
```

Alchemy creates the Project and prints `projectId`. Use the same profile and stage throughout the tutorial so each deployment updates this Stack.

## Next

[Part 2: An HTTP API](part-2.md) adds an Effect-native Compute service to the Project.
