---
url: https://alchemy.run/daytona/sandboxes/programs
title: "Programs in a sandbox"
description: "Run an Effect program you write inside a Daytona sandbox, redeploy it without losing the sandbox, mount volumes into it, and install mount clients with a derived snapshot."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A sandbox can run a program you write, the same way a Worker or a Lambda does. Alchemy bundles it, uploads it into the sandbox, and runs it with Node in a session of its own. The program serves HTTP on a port, reached through the sandbox’s preview URL, and keeps running as long as the sandbox does.

## The sandbox outlives the program

The sandbox outlives the program. A code change restarts the process in the same sandbox, at the same URL. Only a change Daytona can’t make to a running sandbox, like a new mount, replaces it.

<svg viewBox="0 0 662.6 168" style="max-width: 762px" role="img" aria-label="A code change uploads the new bundle and restarts the process in the same sandbox. Adding a mount replaces the sandbox, since Daytona mounts volumes only when it creates one."><line stroke-opacity="0.2" stroke="currentColor" x1="286.6" y1="24" x2="286.6" y2="162"></line><line stroke-opacity="0.2" stroke="currentColor" x1="477.00000000000006" y1="24" x2="477.00000000000006" y2="162"></line><g><text fill="currentColor" x="4" y="56">sandbox 1</text> <g><rect stroke="currentColor" fill="none" x="90.60000000000001" y="40" width="380.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="281" y="56" text-anchor="middle">sandbox.url stays the same</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="662.6" y2="74"></line><text fill="currentColor" x="4" y="100">program</text> <g><rect stroke="currentColor" fill="none" x="90.60000000000001" y="84" width="190.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="185.8" y="100" text-anchor="middle">bundle a1b2</text></g> <g><rect stroke="currentColor" fill="none" x="292.2" y="84" width="179.2" height="24" rx="5"></rect><text fill="currentColor" x="381.79999999999995" y="100" text-anchor="middle">bundle c3d4</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="662.6" y2="118"></line><text fill="currentColor" x="4" y="144">sandbox 2</text> <g><rect stroke="currentColor" fill="none" x="482.6" y="128" width="168" height="24" rx="5"></rect><text fill="currentColor" x="566.6" y="144" text-anchor="middle">new sandbox</text></g></g> <text fill="currentColor" x="286.6" y="18" text-anchor="middle">code change</text> <text fill="currentColor" x="477.00000000000006" y="18" text-anchor="middle">mount added</text></svg> <svg viewBox="0 0 330 198" style="max-width: 379px" role="img" aria-label="A code change uploads the new bundle and restarts the process in the same sandbox. Adding a mount replaces the sandbox, since Daytona mounts volumes only when it creates one."><line stroke-opacity="0.2" stroke="currentColor" x1="111.3" y1="24" x2="111.3" y2="192"></line><line stroke-opacity="0.2" stroke="currentColor" x1="219.42000000000002" y1="24" x2="219.42000000000002" y2="192"></line><g><text fill="currentColor" x="0" y="44">sandbox 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="216.23999999999998" height="24" rx="5"></rect><text fill="currentColor" x="108.11999999999999" y="70" text-anchor="middle">sandbox.url stays the same</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">program</text> <g><rect stroke="currentColor" fill="none" x="0" y="108" width="108.11999999999999" height="24" rx="5"></rect><text fill="currentColor" x="54.059999999999995" y="124" text-anchor="middle">bundle a1b2</text></g> <g><rect stroke="currentColor" fill="none" x="114.47999999999999" y="108" width="101.75999999999999" height="24" rx="5"></rect><text fill="currentColor" x="165.35999999999999" y="124" text-anchor="middle">bundle c3d4</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">sandbox 2</text> <g><rect stroke="currentColor" fill="none" x="222.6" y="162" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="270.3" y="178" text-anchor="middle">new sandbox</text></g></g> <text fill="currentColor" x="111.3" y="18" text-anchor="middle">code change</text> <text fill="currentColor" x="219.42000000000002" y="18" text-anchor="middle">mount added</text></svg>

A code change uploads the new bundle and restarts the process in the same sandbox. Adding a mount replaces the sandbox, since Daytona mounts volumes only when it creates one.

## Run a program

```typescript
import * as Daytona from "@alchemy.run/daytona";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Base } from "./Images.ts";

export default class MySandbox extends Daytona.Sandbox<MySandbox>()(
  "MySandbox",
  { snapshot: Base, main: import.meta.url, port: 3000, public: true },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.json({ ok: true })),
    };
  }),
) {}
```

`sandbox.url` is the preview URL of port 3000. `public` opens it to anyone; a private sandbox’s preview needs its token in `X-Daytona-Preview-Token`. See [Network and previews](../networking/network.md).

## A deploy restarts the process

A code change uploads the new bundle and restarts the process. The sandbox and its URL stay the same.

A sandbox with `main` doesn’t auto-stop. If it stopped anyway, its process ended with it, and the next deploy starts the sandbox and the program again.

## Mount a volume in the program

```typescript
import * as Daytona from "@alchemy.run/daytona";
import * as Effect from "effect/Effect";
import * as FileSystem from "effect/FileSystem";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Base } from "./Images.ts";
import { Data } from "./Storage.ts";

export default class MySandbox extends Daytona.Sandbox<MySandbox>()(
  "MySandbox",
  { snapshot: Base, main: import.meta.url, public: true },
  Effect.gen(function* () {
    const data = yield* Daytona.MountVolume(Data, { path: "/data" });
    const fs = yield* FileSystem.FileSystem;
    return {
      fetch: Effect.gen(function* () {
        return yield* HttpServerResponse.json(yield* fs.readDirectory(data.path));
      }),
    };
  }).pipe(Effect.provide(Daytona.MountVolumeLive)),
) {}
```

The volume joins the sandbox’s `mounts`. Daytona mounts volumes when it creates a sandbox, so adding or removing one replaces the program’s sandbox.

## Install a mount client

A mount binding that needs a client in the image, such as an `alchemy/Mount` capability, makes the sandbox boot from a **derived snapshot**: its snapshot plus the client’s install lines, built on Daytona. The derived snapshot is named from a hash of its Dockerfile and deleted with the sandbox, and a change to it replaces the sandbox.

Daytona sandboxes provide FUSE, so FUSE clients work. A client that needs NFS fails the deploy with `MountUnsupported`.

## The program reaches what its sandbox does

The program reaches the network the way its sandbox does. Below Daytona’s Tier 3, that is only an allowlist of essential services, so bindings that call Daytona’s API work, but calls to another sandbox’s toolbox are refused. From Tier 3, the program reaches the whole internet. See [Network and previews](../networking/network.md).
