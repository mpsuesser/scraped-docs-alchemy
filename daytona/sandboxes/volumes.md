---
url: https://alchemy.run/daytona/sandboxes/volumes
title: "Volumes"
description: "Keep files in volumes that outlive sandboxes, shared or one per user."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **volume** is a persistent disk, backed by object storage, that sandboxes mount at a path. Its files outlive every sandbox that mounts it, and several sandboxes can mount it at once. Daytona mounts volumes only when it creates a sandbox, so a sandbox’s mounts are fixed for its life.

## Files outlive every sandbox

<svg viewBox="0 0 707 102" style="max-width: 813px" role="img" aria-label="Sam's Monday sandbox is deleted, and Friday's new sandbox mounts the same volume at the same path. The files Sam left are still there."><g><text fill="currentColor" x="4" y="34">volume user-sam</text> <g><rect stroke="currentColor" fill="none" x="135" y="18" width="560" height="24" rx="5"></rect><text fill="currentColor" x="415" y="34" text-anchor="middle">/home/daytona/work keeps its files</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="707" y2="52"></line><text fill="currentColor" x="4" y="78">sam's sandboxes</text> <g><rect stroke="currentColor" fill="none" x="163" y="62" width="168" height="24" rx="5"></rect><text fill="currentColor" x="247" y="78" text-anchor="middle">sandbox, Monday</text></g> <g><rect stroke="currentColor" fill="none" x="471" y="62" width="196" height="24" rx="5"></rect><text fill="currentColor" x="569" y="78" text-anchor="middle">sandbox, Friday</text></g></g></svg> <svg viewBox="0 0 330 137" style="max-width: 379px" role="img" aria-label="Sam's Monday sandbox is deleted, and Friday's new sandbox mounts the same volume at the same path. The files Sam left are still there."><g><text fill="currentColor" x="0" y="22">volume user-sam</text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="48" text-anchor="middle">/home/daytona/work keeps its files</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">sam's sandboxes</text> <g><rect stroke="currentColor" fill="none" x="15.9" y="86" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="15.9" y="122" text-anchor="start">sandbox, Monday</text></g> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="86" width="111.29999999999998" height="24" rx="5"></rect><text fill="currentColor" x="246.45" y="102" text-anchor="middle">sandbox, Friday</text></g></g></svg>

Sam's Monday sandbox is deleted, and Friday's new sandbox mounts the same volume at the same path. The files Sam left are still there.

## Declare a shared volume

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Cache = Daytona.Volume("Cache", {});
```

## Mount it in a dev box

```typescript
import * as Daytona from "@alchemy.run/daytona";
import { Base } from "./Images.ts";
import { Cache } from "./Storage.ts";

export const DevBox = Daytona.Sandbox("DevBox", {
  snapshot: Base,
  mounts: [{ volume: Cache, path: "/cache" }],
});
```

## Mount it in sandboxes you create

```typescript
const cache = yield* Daytona.MountVolume(Cache, { path: "/cache" });
const sandbox = yield* sandboxes.create({ mounts: [cache] });
```

In a Worker, `MountVolume` returns a mount entry for the sandboxes you create. In a sandbox that runs a program, it mounts the volume into that sandbox.

## One volume per user

```typescript
const volumes = yield* Daytona.CreateVolume({ prefix: "user-" });
const home = yield* volumes.getOrCreate({ name: \`user-${user}\` });
```

The Worker’s key can read, create and delete volumes. `getOrCreate` waits until a new volume is ready. Volumes have only a name, so `prefix` is how you find the ones the Worker created: the binding refuses names outside it. They are not deleted when the stack is destroyed; `volumes.list()` and delete the names with your prefix to clean them up.

## Delete a volume

```typescript
yield* volumes.delete(home.volumeId);
```

Deleting a volume deletes its files. A volume can’t be deleted while a sandbox mounts it, and Daytona releases it about a minute after that sandbox is deleted, so `delete` fails with `VolumeInUse` until then: retry later rather than holding a request open.

## Write new files, don’t append

A volume is backed by object storage, so writing a whole file works but appending to an existing one doesn’t. Write a new file instead of `>>` onto an old one, like a file per visit or per run.

When you destroy the stack, a declared `Daytona.Volume` waits for the sandboxes that mount it to go first.
