---
url: https://alchemy.run/archil/tutorial/part-1
title: "Part 1: Set Up a Sandbox"
description: "Install Alchemy, and declare an Archil sandbox that clones a repository onto its disk at deploy time."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Build an API that gives agents sandboxes. The slow setup runs once, when you deploy: a sandbox with a repository cloned onto its disk. While the app runs, every sandbox is a fork of it, already set up.

The finished code is in [`examples/archil-sandbox`](https://github.com/alchemy-run/alchemy/tree/main/examples/archil-sandbox).

## Prerequisites

- [Node.js](https://nodejs.org/) 24.11.1+ or [Bun](https://bun.sh/)
- An Archil API key in an Alchemy profile ([Setup](../setup.md))

## Create a project

```sh
mkdir my-app && cd my-app && pnpm init
```

```sh
pnpm add alchemy@latest @alchemy.run/archil@latest effect @effect/platform-node
```

## Declare the sandbox

```typescript
import * as Archil from "@alchemy.run/archil";

export const Golden = Archil.Sandbox("Golden", {
  run: ["git clone --depth 1 https://github.com/sindresorhus/is /work/is"],
});
```

A sandbox is a Firecracker microVM whose root filesystem is a disk of its own. At deploy, Alchemy creates `Golden` and runs the `run` commands in it, in order. A command that exits non-zero fails the deploy.

## Create the Stack

```typescript
import * as Archil from "@alchemy.run/archil";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import { Golden } from "./src/Golden.ts";

export default Alchemy.Stack(
  "ArchilSandboxExample",
  { providers: Archil.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const golden = yield* Golden;
    return { sandbox: golden.name };
  }),
);
```

`Archil.providers()` reads your key from your Alchemy profile.

## Deploy

```sh
pnpm alchemy deploy
```

`Golden` shows up under **Sandboxes** in the [Archil console](https://console.archil.com/), with the repository in `/work/is`. A deploy that leaves the commands unchanged leaves it alone. When it’s idle, it pauses.

## Next

[Part 2](part-2.md) forks it while the app runs.
