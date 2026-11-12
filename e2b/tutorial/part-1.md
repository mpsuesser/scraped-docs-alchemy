---
url: https://alchemy.run/e2b/tutorial/part-1
title: "Part 1: Build a Template and a Snapshot"
description: "Declare an E2B template and a snapshot of it with a repository cloned, both built at deploy time, so every sandbox starts ready."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Build an API that gives agents sandboxes. The slow setup runs once, when you deploy: an image with the tools, and a snapshot of it with a repository cloned. While the app runs, every sandbox starts from that snapshot in well under a second, already set up.

The finished code is in [`examples/e2b-agent-sandbox`](https://github.com/alchemy-run/alchemy/tree/main/examples/e2b-agent-sandbox).

## Prerequisites

Finish [E2B setup](../setup.md) and [Cloudflare setup](../../cloudflare/setup.md).

## Declare the template

A template is the image a sandbox boots from:

```typescript
import * as Dockerfile from "@alchemy.run/docker/Dockerfile";
import * as E2B from "@alchemy.run/e2b";

export const MyTemplate = E2B.Template("MyTemplate", {
  dockerfile: Dockerfile.inline\`
    FROM node:22-bookworm
    RUN apt-get update && apt-get install -y git
  \`,
  cpuCount: 2,
  memoryMB: 2048,
});
```

E2B builds it on its side, so you don’t need Docker.

## Snapshot it with the repository cloned

A snapshot starts a sandbox from the template, runs commands in it, and saves the result:

```typescript
export const Warm = E2B.Snapshot("Warm", {
  template: MyTemplate,
  run: ["git clone --depth 1 https://github.com/sindresorhus/is repo"],
});
```

Every sandbox the app starts boots from `Warm`, with the repository already in `/home/user/repo`.

## Add it to the Stack

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as E2B from "@alchemy.run/e2b";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { Warm } from "./src/Sandbox.ts";

export default Alchemy.Stack(
  "E2BAgentSandboxExample",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), E2B.providers()),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const warm = yield* Warm;
    return { snapshot: warm.templateId };
  }),
);
```

`Warm` depends on `MyTemplate`, so deploying it builds both.

## Deploy

```sh
pnpm alchemy deploy
```

The first build takes a minute or two. Later deploys rebuild only what changed.

## Next

[Part 2](part-2.md) starts sandboxes from it while the app runs.
