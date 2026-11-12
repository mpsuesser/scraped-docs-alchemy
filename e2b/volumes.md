---
url: https://alchemy.run/e2b/volumes
title: "Volumes"
description: "Keep files that outlive sandboxes, mount them by name, and read or write them without a sandbox."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **volume** is a filesystem that lives outside any sandbox. A sandbox mounts it at a path when it’s created, and what it writes there stays after the sandbox is killed, for the next sandbox that mounts it. You can also read and write a volume’s files without a sandbox.

## Files outlive the sandbox

<svg viewBox="0 0 647.8 146" style="max-width: 745px" role="img" aria-label="Each build sandbox mounts Cache at /cache. The first fills it and is killed; the files stay on the volume, and the next build starts with them."><g><text fill="currentColor" x="4" y="28">Cache</text> <text fill="currentColor" x="4" y="42">volume</text> <g><rect stroke="currentColor" fill="none" x="75.80000000000001" y="18" width="560" height="24" rx="5"></rect><text fill="currentColor" x="355.8" y="34" text-anchor="middle">files in /cache</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="647.8" y2="52"></line><text fill="currentColor" x="4" y="72">build 1</text> <text fill="currentColor" x="4" y="86">sandbox</text> <g><rect stroke="currentColor" fill="none" x="131.8" y="62" width="168" height="24" rx="5"></rect><text fill="currentColor" x="215.8" y="78" text-anchor="middle">fills /cache</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="647.8" y2="96"></line><text fill="currentColor" x="4" y="116">build 2</text> <text fill="currentColor" x="4" y="130">sandbox</text> <g><rect stroke="currentColor" fill="none" x="411.8" y="106" width="167.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="495.79999999999995" y="122" text-anchor="middle">reuses /cache</text></g></g></svg> <svg viewBox="0 0 330 176" style="max-width: 379px" role="img" aria-label="Each build sandbox mounts Cache at /cache. The first fills it and is killed; the files stay on the volume, and the next build starts with them."><g><text fill="currentColor" x="0" y="22">Cache <tspan fill="currentColor" dx="8">volume</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="48" text-anchor="middle">files in /cache</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">build 1 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="31.8" y="86" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="79.5" y="102" text-anchor="middle">fills /cache</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">build 2 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="140" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="238.5" y="156" text-anchor="middle">reuses /cache</text></g></g></svg>

Each build sandbox mounts Cache at /cache. The first fills it and is killed; the files stay on the volume, and the next build starts with them.

E2B enables volumes per team. On a team without them, deploying a volume fails with `VolumesNotEnabled`.

## Declare a shared volume

```typescript
import * as E2B from "@alchemy.run/e2b";

export const Cache = E2B.Volume("Cache", {});
```

Changing `name` replaces the volume and loses its files.

## Mount it into a declared sandbox

```typescript
import * as E2B from "@alchemy.run/e2b";
import { MyTemplate } from "./MyTemplate.ts";
import { Cache } from "./Storage.ts";

export const Builder = E2B.Sandbox("Builder", {
  template: MyTemplate,
  mounts: [{ volume: Cache, path: "/cache" }],
});
```

E2B attaches volumes only when a sandbox is created, so changing `mounts` replaces the sandbox.

## Mount it from a Worker

```typescript
// in the Worker's construction
const cache = yield* E2B.MountVolume(Cache, { path: "/cache" });
const sandboxes = yield* E2B.CreateSandbox(MyTemplate);
// in fetch
const sandbox = yield* sandboxes.create({ mounts: [cache] });
```

`MountVolume` returns a reference with the entry’s fields, so it goes straight into `mounts`. Provide `E2B.MountVolumeLive`.

## Mount it in a template program

```typescript
import * as E2B from "@alchemy.run/e2b";
import * as Effect from "effect/Effect";
import { FileSystem } from "effect/FileSystem";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Cache } from "./Storage.ts";

export default class Api extends E2B.Template<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const cache = yield* E2B.MountVolume(Cache, { path: "/cache" });
    const fs = yield* FileSystem;
    return {
      fetch: Effect.gen(function* () {
        const files = yield* fs.readDirectory(cache.path);
        return yield* HttpServerResponse.json(files);
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(E2B.MountVolumeLive)),
) {}
```

Inside a template program, the mount becomes part of the template. Every sandbox created from it, with `CreateSandbox(Api)` or `E2B.Sandbox({ template: Api })`, mounts the volume.

## Create a volume per user

```typescript
const volumes = yield* E2B.CreateVolume();
// in fetch:
const home = yield* volumes.getOrCreate(\`home-${user}\`);
const sandbox = yield* sandboxes.create({ mounts: [{ volume: home, path: "/home/user/work" }] });
```

Provide `E2B.CreateVolumeHttp`. Runtime volumes are not deleted when you destroy the Worker: call `volumes.delete(home.volumeId)` when the user goes away.

## Read files without a sandbox

```typescript
const reports = yield* E2B.ReadVolume(Reports);
// in fetch:
const report = yield* reports.readText("/report.json");
const entries = yield* reports.list("/");
```

Provide `E2B.ReadVolumeHttp`. The client uses the volume’s own token, looked up with the host’s API key.

## Write files without a sandbox

```typescript
const inputs = yield* E2B.WriteVolume(Inputs);
// in fetch:
yield* inputs.write("/task.json", JSON.stringify(task));
yield* inputs.makeDir("/results");
yield* inputs.remove("/old");
```

`E2B.ReadWriteVolume` does both. A runtime volume’s `files` is a read-write client too.
