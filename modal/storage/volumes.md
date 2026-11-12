---
url: https://alchemy.run/modal/storage/volumes
title: "Volumes"
description: "Keep files on Modal volumes and read or write them without a sandbox."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **Volume** is a filesystem in an environment that Functions, Servers and Sandboxes mount at a path. Containers come and go; the Volume keeps their files, and a Worker can read and write them without any container at all.

## The files outlive every container

<svg viewBox="0 0 662.6 146" style="max-width: 762px" role="img" aria-label="Session 1 writes a file to the Volume and stops. Session 2 mounts the same Volume later and finds it there."><g><text fill="currentColor" x="4" y="28">home-sam</text> <text fill="currentColor" x="4" y="42">Volume</text> <g><rect stroke="currentColor" fill="none" x="90.60000000000001" y="18" width="560" height="24" rx="5"></rect><text fill="currentColor" x="370.6" y="34" text-anchor="middle">/home/user</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="662.6" y2="52"></line><text fill="currentColor" x="4" y="72">session 1</text> <text fill="currentColor" x="4" y="86">Sandbox</text> <g><rect stroke="currentColor" fill="none" x="146.60000000000002" y="62" width="168" height="24" rx="5"></rect><text fill="currentColor" x="230.60000000000002" y="78" text-anchor="middle">writes notes.md</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="662.6" y2="96"></line><text fill="currentColor" x="4" y="116">session 2</text> <text fill="currentColor" x="4" y="130">Sandbox</text> <g><rect stroke="currentColor" fill="none" x="426.6" y="106" width="168" height="24" rx="5"></rect><text fill="currentColor" x="510.6" y="122" text-anchor="middle">reads notes.md</text></g></g></svg> <svg viewBox="0 0 330 191" style="max-width: 379px" role="img" aria-label="Session 1 writes a file to the Volume and stops. Session 2 mounts the same Volume later and finds it there."><g><text fill="currentColor" x="0" y="22">home-sam <tspan fill="currentColor" dx="8">Volume</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="48" text-anchor="middle">/home/user</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">session 1 <tspan fill="currentColor" dx="8">Sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="31.8" y="86" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="133.2" y="102" text-anchor="start">writes notes.md</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">session 2 <tspan fill="currentColor" dx="8">Sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="140" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="190.79999999999998" y="176" text-anchor="start">reads notes.md</text></g></g></svg>

Session 1 writes a file to the Volume and stops. Session 2 mounts the same Volume later and finds it there.

## Declare a Volume

```typescript
import * as Modal from "@alchemy.run/modal";

export const Models = Modal.Volume("Models", {});
```

## Mount it into a Function, Server or program

A `mounts` entry is a volume and a path. Every container the program runs in sees the volume there.

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as FileSystem from "effect/FileSystem";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Backend, Models } from "./Storage.ts";

export default class Embed extends Modal.Function<Embed>()(
  "Embed",
  { main: import.meta.url, app: Backend, mounts: [{ volume: Models, path: "/models", readOnly: true }] },
  Effect.gen(function* () {
    const fs = yield* FileSystem.FileSystem;
    return {
      fetch: Effect.gen(function* () {
        return yield* HttpServerResponse.json(yield* fs.readDirectory("/models"));
      }).pipe(Effect.orDie),
    };
  }),
) {}
```

`Modal.MountVolume(Models, { path: "/models" })` does the same from the program itself, next to its other bindings.

## Mount it into a sandbox you create

In a Worker or Lambda, `Modal.MountVolume` returns the entry for `sandboxes.create`:

```typescript
// construction phase
const models = yield* Modal.MountVolume(Models, { path: "/models", readOnly: true });
const sandboxes = yield* Modal.CreateSandbox(Coder);

// per request
const sandbox = yield* sandboxes.create({ mounts: [models] });
```

A volume created at runtime mounts the same way: `{ volume: home, path: "/home/user" }`.

## Mount a cloud bucket

A `bucket` entry mounts an S3, R2 or GCS bucket. Pass a bucket name, or a bucket resource of another provider; a Cloudflare R2 bucket’s account endpoint is filled in. `secret` is a `Modal.Secret` holding the bucket’s keys.

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as FileSystem from "effect/FileSystem";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Coder, R2Keys } from "./Storage.ts";

export const Uploads = Cloudflare.R2.Bucket("Uploads", {});

export default class MySandbox extends Modal.Sandbox<MySandbox>()(
  "MySandbox",
  { main: import.meta.url, app: Coder, mounts: [{ bucket: Uploads, path: "/uploads", secret: R2Keys }] },
  Effect.gen(function* () {
    const fs = yield* FileSystem.FileSystem;
    return {
      fetch: Effect.gen(function* () {
        return yield* HttpServerResponse.json(yield* fs.readDirectory("/uploads"));
      }).pipe(Effect.orDie),
    };
  }),
) {}
```

`Modal.MountBucket(Uploads, { path: "/uploads", secret: R2Keys })` is the binding form.

## Read and write from a Worker

Bind `ReadVolume`, `WriteVolume` or `ReadWriteVolume`:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Models } from "./Storage.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url, compatibility: { flags: ["nodejs_compat"] } },
  Effect.gen(function* () {
    const models = yield* Modal.ReadWriteVolume(Models);
    return {
      fetch: Effect.gen(function* () {
        yield* models.writeFile("/hello.txt", "hello");
        const text = yield* models.readFileString("/hello.txt");
        return HttpServerResponse.text(text);
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(Modal.ReadWriteVolumeHttp)),
) {}
```

## List, copy and remove files

```typescript
const files = yield* models.listFiles("/", { recursive: true });
yield* models.copyFiles(["/hello.txt"], "/backup/hello.txt");
yield* models.removeFile("/backup", { recursive: true });
```

## One volume per user

```typescript
const volumes = yield* Modal.CreateVolume(MyEnv);
const home = yield* volumes.getOrCreate(\`home-${user}\`);
yield* home.writeFile("/notes.md", notes);
```

Objects created at run time are tagged with the stack, stage and binding (`alchemy::stack`, `alchemy::stage`, `alchemy::id`) and are not deleted when the stack is destroyed. A volume has only a name, so the tags become its name prefix. Find them by that prefix to clean them up:

```typescript
const prefix = Modal.runtimePrefix({ stack: "my-app", stage: "prod", id: "Api" });
```

```sh
modal volume list --env "$ENV" | grep "$PREFIX"
```
