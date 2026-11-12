---
url: https://alchemy.run/docker/volumes
title: "Volumes"
description: "Keep a container's data in a Docker.Volume so it survives the container being recreated, mount host directories, and know when a volume and its data are deleted."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A container’s own filesystem is removed with the container, and most changes to a container [recreate it](containers.md#what-a-change-does). A **volume** is storage the engine keeps outside any container. Mount it at a path, and whatever is written there stays on the volume when the container goes, ready for the next one.

<svg viewBox="0 0 692.2 124" style="max-width: 796px" role="img" aria-label="Changing the image removes the container and creates a new one. The volume outlives both, so the new container starts on the old data."><line stroke-opacity="0.2" stroke="currentColor" x1="377.79999999999995" y1="24" x2="377.79999999999995" y2="118"></line><g><text fill="currentColor" x="4" y="50">postgres</text> <text fill="currentColor" x="4" y="64">container</text> <g><rect stroke="currentColor" fill="none" x="120.2" y="40" width="257.59999999999997" height="24" rx="5"></rect><text fill="currentColor" x="249" y="56" text-anchor="middle">postgres:17</text></g> <g><rect stroke="currentColor" fill="none" x="411.4" y="40" width="268.80000000000007" height="24" rx="5"></rect><text fill="currentColor" x="545.8" y="56" text-anchor="middle">postgres:18 · recreated</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="692.2" y2="74"></line><text fill="currentColor" x="4" y="94">postgres-data</text> <text fill="currentColor" x="4" y="108">volume</text> <g><rect stroke="currentColor" fill="none" x="120.2" y="84" width="560" height="24" rx="5"></rect><text fill="currentColor" x="400.2" y="100" text-anchor="middle">/var/lib/postgresql/data</text></g></g> <text fill="currentColor" x="377.79999999999995" y="18" text-anchor="middle">image changed</text></svg> <svg viewBox="0 0 330 159" style="max-width: 379px" role="img" aria-label="Changing the image removes the container and creates a new one. The volume outlives both, so the new container starts on the old data."><line stroke-opacity="0.2" stroke="currentColor" x1="146.28" y1="24" x2="146.28" y2="153"></line><g><text fill="currentColor" x="0" y="44">postgres <tspan fill="currentColor" dx="8">container</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="146.28" height="24" rx="5"></rect><text fill="currentColor" x="73.14" y="70" text-anchor="middle">postgres:17</text></g> <g><rect stroke="currentColor" fill="none" x="165.36" y="54" width="152.64" height="24" rx="5"></rect><text fill="currentColor" x="165.36" y="90" text-anchor="start">postgres:18 · recreated</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="99" x2="330" y2="99"></line><text fill="currentColor" x="0" y="113">postgres-data <tspan fill="currentColor" dx="8">volume</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="123" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="139" text-anchor="middle">/var/lib/postgresql/data</text></g></g> <text fill="currentColor" x="146.28" y="18" text-anchor="middle">image changed</text></svg>

Changing the image removes the container and creates a new one. The volume outlives both, so the new container starts on the old data.

## Mount a volume

```typescript
import * as Alchemy from "alchemy";
import * as Docker from "@alchemy.run/docker";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Docker.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const data = yield* Docker.Volume("postgres-data");

    yield* Docker.Container("postgres", {
      image: "postgres:18-alpine",
      environment: { POSTGRES_PASSWORD: yield* Config.Redacted("POSTGRES_PASSWORD") },
      volumes: [{ hostPath: data.name, containerPath: "/var/lib/postgresql/data" }],
      start: true,
    });

    return { volume: data.name };
  }),
);
```

`hostPath` set to a volume’s name mounts that volume. Several containers can mount one volume, and `readOnly: true` mounts it read-only.

## Mount a directory from the host

When `hostPath` is a path, Docker mounts that directory instead, a **bind mount**:

```typescript
const web = yield* Docker.Container("web", {
  image: "nginx:alpine",
  volumes: [{ hostPath: "/srv/site", containerPath: "/usr/share/nginx/html", readOnly: true }],
  ports: [{ external: 8080, internal: 80 }],
  start: true,
});
```

The path is on the engine’s machine, which on a remote [context](engines-and-contexts.md) is the server, not your laptop. Docker Desktop shares only the folders listed in its file sharing settings.

## Store it elsewhere

A volume uses the `local` driver by default, a directory on the engine’s disk (its `mountpoint` attribute). `driverOpts` passes options to the driver, such as mounting an NFS share:

```typescript
const shared = yield* Docker.Volume("shared", {
  driverOpts: { type: "nfs", o: "addr=10.0.0.1,rw", device: ":/exports/shared" },
});
```

## When the data is deleted

Destroying a `Docker.Volume` removes it, and its data with it. So does changing its `name`, `context`, `driver`, `driverOpts` or `labels`, which replaces the volume with a new, empty one. Treat a volume’s props as fixed once it holds data you care about.

To keep a volume when the Stack is destroyed, retain it. Alchemy then forgets it instead of removing it:

```typescript
import * as RemovalPolicy from "alchemy/RemovalPolicy";

const data = yield* Docker.Volume("postgres-data").pipe(RemovalPolicy.retain());
```

See [Removal policy](../infrastructure-as-code/resource-lifecycle.md#removal-policy). For a volume created outside the Stack, look it up. Alchemy reads it and never removes it:

```typescript
const data = Docker.Volume.lookup({ name: "pgdata" });
```

## A volume with the same name

Docker names are the only handle on a volume, so Alchemy labels each volume it creates with a generation ID and checks it before every change. If the volume was removed and another one now has its name, plan, deploy and destroy fail with `OwnedBySomeoneElse` and leave that volume alone.

To manage a volume created outside the Stack, adopt it:

```typescript
import { adopt } from "alchemy/AdoptPolicy";

const data = yield* Docker.Volume("postgres-data", { name: "pgdata" }).pipe(adopt(true));
```

Alchemy never deletes an adopted volume to apply a changed `driver`, `driverOpts` or `labels`; the plan fails and you migrate the data yourself. Docker removes volumes by name, so a volume swapped in between the check and the removal can still be deleted. For data you can’t lose, use `RemovalPolicy.protect()`, which refuses both replacement and destroy.
