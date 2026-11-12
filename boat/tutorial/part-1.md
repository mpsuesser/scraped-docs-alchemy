---
url: https://alchemy.run/boat/tutorial/part-1
title: "Part 1: Build a Snapshot"
description: "Declare a Boat snapshot that clones a repository and installs tools at deploy time, so every sandbox starts ready."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Build an API that gives agents sandboxes. The slow setup runs once, when you deploy: a snapshot with a repository cloned and its tools installed. While the app runs, every sandbox starts from that snapshot in under a second, already set up.

The finished code is in [`examples/boat-agent-sandbox`](https://github.com/alchemy-run/alchemy/tree/main/examples/boat-agent-sandbox).

## Prerequisites

Finish [Boat setup](../setup.md) and [Cloudflare setup](../../cloudflare/setup.md).

## Declare the snapshot

A `Boat.Snapshot` lists the commands that set it up:

```typescript
import * as Boat from "@alchemy.run/boat";

export const Golden = Boat.Snapshot("Golden", {
  run: ["git clone --depth 1 https://github.com/sindresorhus/is ~/repo"],
});
```

At deploy, Alchemy starts a sandbox with none of your account’s secrets, runs the commands in order, saves its disk as `Golden` and deletes the sandbox.

## Add it to the Stack

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { Golden } from "./src/Golden.ts";

export default Alchemy.Stack(
  "BoatAgentSandboxExample",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), Boat.providers()),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const golden = yield* Golden;
    return { snapshot: golden.name };
  }),
);
```

## Deploy

```sh
pnpm alchemy deploy
```

The deploy takes as long as the clone. The snapshot is in your Boat dashboard. A deploy that leaves the commands unchanged leaves it alone.

## Next

[Part 2](part-2.md) starts sandboxes from it while the app runs.
