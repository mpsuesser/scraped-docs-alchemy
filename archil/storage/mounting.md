---
url: https://alchemy.run/archil/storage/mounting
title: "Mounting"
description: "Mount Archil disks in program sandboxes, your own Containers, and ECS tasks."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

`Archil.MountDisk(disk, { path, mode })` returns a mount reference: the fields of a sandbox `mounts` entry, plus `mount()`. Where it mounts depends on the host it’s bound to.

## Mount it in a program sandbox

An [`Archil.Sandbox` with `main`](../compute/programs.md) mounts the disk itself. The binding adds it to the sandbox’s mounts, and the disk is there when the program starts:

```typescript
const dataset = yield* Archil.MountDisk(Dataset, { path: "/data", mode: "read-only" });
const names = yield* fs.readDirectory(dataset.path);
```

## Mount it on your own hosts

On a Cloudflare Container or an ECS task on EC2, the binding installs the Archil client into the host’s image, and the plan fails if the host can’t provide FUSE. `mount()` runs the client:

```typescript
export default class Indexer extends Cloudflare.Container<Indexer>()(
  "Indexer",
  { main: import.meta.url },
  Effect.gen(function* () {
    const dataset = yield* Archil.MountDisk(Dataset, { path: "/data", mode: "read-only" });
    const fs = yield* FileSystem.FileSystem;
    return {
      fetch: Effect.scoped(
        Effect.gen(function* () {
          const path = yield* dataset.mount();
          const entries = yield* fs.readDirectory(path);
          return yield* HttpServerResponse.json(entries);
        }),
      ),
    };
  }).pipe(Effect.provide(Archil.MountDiskLive)),
) {}
```

The mount lasts as long as its scope. Closing the scope flushes writes and unmounts.

The host gets a disk token for `Dataset` only, so it can’t mount any other disk and never holds an API key. With `mode: "read-only"` the token is read-only too.

## Start the Container with internet access

The Archil client talks to Archil over the network, and a Cloudflare Container has no outbound network unless the Durable Object that starts it asks for one:

```typescript
export class IndexerObject extends Cloudflare.DurableObject<IndexerObject>()(
  "IndexerObject",
  Effect.gen(function* () {
    const indexer = yield* Indexer;
    return Effect.gen(function* () {
      const { fetch } = yield* indexer.getTcpPort(3000);
      return {
        list: () =>
          Effect.gen(function* () {
            yield* indexer.start({ enableInternet: true });
            const response = yield* fetch(HttpClientRequest.get("http://container/"));
            return yield* response.text;
          }),
      };
    });
  }),
) {}
```

Without `enableInternet: true`, `mount` fails after 60 seconds with the client’s output.

## Mount disks named at run time

`MountDisk()` takes the disk and its token per call, e.g. from a Worker that created the disk:

```typescript
const archil = yield* Archil.MountDisk();
const path = yield* archil.mount({ disk: job.diskId, token: job.token, path: "/work" });
```

The Worker makes the token with `home.createToken(name)` and revokes it with `home.removeToken(id)`.

`mode` picks how this mount shares the disk. An `exclusive` mount, the default, locks the whole disk for one writer, and every other client can only mount it read-only:

<svg viewBox="0 0 599.8 230" style="max-width: 690px" role="img" aria-label="The Indexer holds the disk's write lock, so the Api only reads it and the Job can't write until the Indexer unmounts. Mount both writers conditional and both can write."><defs><marker id="alc-bd-arrow-h4gmen" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="213.79999999999998" y="118" width="172.20000000000002" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="152.2" y1="172" x2="227.79999999999998" y2="172" marker-end="url(#alc-bd-arrow-h4gmen)"></line><text fill="currentColor" x="192" y="164" text-anchor="middle">writes</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="583.8" y1="172" x2="227.79999999999998" y2="172" marker-end="url(#alc-bd-arrow-h4gmen)"></line><text fill="currentColor" x="407.79999999999995" y="164" text-anchor="middle">reads</text></g> <g><path stroke="currentColor" fill="none" d="M 368 40 H 299.9 V 114"></path><path stroke="currentColor" fill="none" d="M 292.9 111 l 14 14 m 0 -14 l -14 14"></path></g><text fill="currentColor" x="374" y="137" text-anchor="end">Dataset <tspan fill="currentColor" dx="8">disk</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="84.1" y="169" text-anchor="middle">Indexer</text> <text fill="currentColor" x="84.1" y="185" text-anchor="middle">exclusive</text></g> <g><rect stroke="currentColor" fill="none" x="231.79999999999998" y="16" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="299.9" y="37" text-anchor="middle">Job</text> <text fill="currentColor" x="299.9" y="53" text-anchor="middle">exclusive</text></g> <g><rect stroke="currentColor" fill="none" x="447.6" y="148" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="515.7" y="169" text-anchor="middle">Api</text> <text fill="currentColor" x="515.7" y="185" text-anchor="middle">read-only</text></g> <g><rect stroke="currentColor" fill="none" x="231.79999999999998" y="148" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="299.9" y="169" text-anchor="middle">/</text> <text fill="currentColor" x="299.9" y="185" text-anchor="middle">locked by Indexer</text></g></svg> <svg viewBox="0 0 370.4 376" style="max-width: 426px" role="img" aria-label="The Indexer holds the disk's write lock, so the Api only reads it and the Job can't write until the Indexer unmounts. Mount both writers conditional and both can write."><defs><marker id="alc-bd-arrow-dav6wa" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="182.2" y="134" width="172.2" height="140" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="268.29999999999995" y1="64" x2="268.29999999999995" y2="160" marker-end="url(#alc-bd-arrow-dav6wa)"></line><text fill="currentColor" x="278.29999999999995" y="118">writes</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="268.29999999999995" y1="360" x2="268.29999999999995" y2="160" marker-end="url(#alc-bd-arrow-dav6wa)"></line><text fill="currentColor" x="278.29999999999995" y="266">reads</text></g> <g><path stroke="currentColor" fill="none" d="M 84.1 212 V 258 H 178.2"></path><path stroke="currentColor" fill="none" d="M 173.2 251 l 12 14 m 0 -14 l -12 14"></path></g><text fill="currentColor" x="342.4" y="153" text-anchor="end">Dataset <tspan fill="currentColor" dx="8">disk</tspan></text> <g><rect stroke="currentColor" fill="none" x="200.2" y="16" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="268.29999999999995" y="37" text-anchor="middle">Indexer</text> <text fill="currentColor" x="268.29999999999995" y="53" text-anchor="middle">exclusive</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="164" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="84.1" y="185" text-anchor="middle">Job</text> <text fill="currentColor" x="84.1" y="201" text-anchor="middle">exclusive</text></g> <g><rect stroke="currentColor" fill="none" x="200.2" y="312" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="268.29999999999995" y="333" text-anchor="middle">Api</text> <text fill="currentColor" x="268.29999999999995" y="349" text-anchor="middle">read-only</text></g> <g><rect stroke="currentColor" fill="none" x="200.2" y="164" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="268.29999999999995" y="185" text-anchor="middle">/</text> <text fill="currentColor" x="268.29999999999995" y="201" text-anchor="middle">locked by Indexer</text></g></svg>

The Indexer holds the disk's write lock, so the Api only reads it and the Job can't write until the Indexer unmounts. Mount both writers conditional and both can write.

```typescript
const dataset = yield* Archil.MountDisk(Dataset, { path: "/data", mode: "shared" });
```

`exclusive` (the default) owns writes; `shared` lets many clients mount and write only paths they check out; `conditional` lets many clients write anywhere; `read-only` never writes. Mount a branch with `branch: "session-42"`. `shared`, `branch`, and `force` are FUSE-client options: a sandbox mounts with `exclusive`, `conditional`, or `read-only`, and takes `queueMs`.

## Pass a mount to a sandbox

A mount reference has the same fields as a sandbox `mounts` entry, so it goes straight into a sandbox created at run time:

```typescript
const workspace = yield* Archil.MountDisk(Workspace, { path: "/workspace", mode: "conditional" });
const sandboxes = yield* Archil.CreateSandbox();
const sandbox = yield* sandboxes.create({ name: \`task-${id}\`, mounts: [workspace] });
```

## Mount it in an ECS task

ECS tasks need the EC2 launch type, since Fargate has no FUSE:

```typescript
export default class Reader extends AWS.ECS.Task<Reader>()(
  "Reader",
  { main: import.meta.url, port: 3000, requiresCompatibilities: ["EC2"], networkMode: "bridge" },
  Effect.gen(function* () {
    const dataset = yield* Archil.MountDisk(Dataset, { path: "/data", mode: "read-only" });
    const fs = yield* FileSystem.FileSystem;
    return {
      fetch: Effect.scoped(
        Effect.gen(function* () {
          const path = yield* dataset.mount();
          const entries = yield* fs.readDirectory(path);
          return yield* HttpServerResponse.json(entries);
        }),
      ),
    };
  }).pipe(Effect.provide(Archil.MountDiskLive)),
) {}
```

Alchemy adds `/dev/fuse` and `SYS_ADMIN` to the task’s container.
