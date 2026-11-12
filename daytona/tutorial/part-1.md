---
url: https://alchemy.run/daytona/tutorial/part-1
title: "Part 1: Build a Warm Snapshot"
description: "Declare a Daytona snapshot of an image, and a warm snapshot of it with a repository cloned, both built at deploy time."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Build an API that gives agents sandboxes. The slow setup runs once, when you deploy: an image pulled into a snapshot, and a second snapshot of it with a repository cloned. While the app runs, every sandbox starts from that snapshot, already set up.

The finished code is in [`examples/daytona-sandboxes`](https://github.com/alchemy-run/alchemy/tree/main/examples/daytona-sandboxes).

## Prerequisites

Finish [Daytona setup](../setup.md) and [Cloudflare setup](../../cloudflare/setup.md).

## Pull an image into a snapshot

A snapshot is the image a sandbox boots from, with its size:

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Base = Daytona.Snapshot("Base", {
  image: "node:22-bookworm",
  cpu: 1,
  memory: 1,
  disk: 3,
});
```

Daytona pulls `node:22-bookworm` once, with Node.js and git.

## Capture a warm snapshot with the repository cloned

```typescript
export const Warm = Daytona.Snapshot("Warm", {
  from: Base,
  run: ["git clone --depth 1 https://github.com/sindresorhus/is /home/daytona/repo"],
});
```

At deploy, Alchemy starts a sandbox from `Base`, runs the commands, captures its disk as `Warm`, and deletes the sandbox. Every sandbox the app starts boots from it, with the repository already in `/home/daytona/repo`.

## Add it to the Stack

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Daytona from "@alchemy.run/daytona";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { Warm } from "./src/Images.ts";

export default Alchemy.Stack(
  "DaytonaSandboxes",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), Daytona.providers()),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const warm = yield* Warm;
    return { snapshot: warm.name };
  }),
);
```

`Warm` starts from `Base`, so deploying it builds both.

## Deploy

```sh
pnpm alchemy deploy
```

Both snapshots show up under **Snapshots** in the Daytona dashboard once they’re active. A deploy that changes neither leaves them alone.

## Next

[Part 2](part-2.md) starts sandboxes from it while the app runs.
