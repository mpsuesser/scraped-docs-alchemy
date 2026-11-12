---
url: https://alchemy.run/fly/compute/sprites
title: "Sprites"
description: "An Effect program running in a Fly.io Sprite. It hibernates when idle."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A [`Sprite`](https://alchemy.run/providers/fly/reference/sprite#sprite) is an Effect program running in a Fly.io Sprite. Sprites are org-scoped Linux sandboxes. They hibernate when idle and wake on demand.

A [`Service`](services.md) is an always-on Machine on an [App](apps.md). A Sprite has no parent App. Alchemy bundles `main` and writes it onto the Sprite. There is no Docker image.

Auth is `FLY_API_TOKEN`. Alchemy mints a Sprites bearer from that token. There is no extra token to store.

<svg viewBox="0 0 672.8 80" style="max-width: 774px" role="img" aria-label="A Sprite hibernates when nothing uses it and wakes on the next request, so an idle sandbox costs little."><line stroke-opacity="0.2" stroke="currentColor" x1="414.4" y1="24" x2="414.4" y2="74"></line><g><text fill="currentColor" x="4" y="50">Sprite</text> <text fill="currentColor" x="4" y="64">your program</text> <g><rect stroke="currentColor" fill="none" x="100.80000000000001" y="40" width="123.20000000000002" height="24" rx="5"></rect><text fill="currentColor" x="162.40000000000003" y="56" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="224.00000000000003" y="40" width="190.39999999999995" height="24" rx="5"></rect><text fill="currentColor" x="319.2" y="56" text-anchor="middle">hibernated · idle</text></g> <g><rect stroke="currentColor" fill="none" x="414.4" y="40" width="28" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="442.4" y="40" width="106.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="495.59999999999997" y="56" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="548.8" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="604.8" y="56" text-anchor="middle">hibernated</text></g></g> <text fill="currentColor" x="414.4" y="18" text-anchor="middle">a request wakes it</text></svg> <svg viewBox="0 0 330 105" style="max-width: 379px" role="img" aria-label="A Sprite hibernates when nothing uses it and wakes on the next request, so an idle sandbox costs little."><line stroke-opacity="0.2" stroke="currentColor" x1="178.07999999999998" y1="24" x2="178.07999999999998" y2="99"></line><g><text fill="currentColor" x="0" y="44">Sprite <tspan fill="currentColor" dx="8">your program</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="69.96000000000001" height="24" rx="5"></rect><text fill="currentColor" x="34.980000000000004" y="70" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="69.96000000000001" y="54" width="108.11999999999998" height="24" rx="5"></rect><text fill="currentColor" x="69.96000000000001" y="90" text-anchor="start">hibernated · idle</text></g> <g><rect stroke="currentColor" fill="none" x="178.07999999999998" y="54" width="15.900000000000006" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="193.98" y="54" width="60.420000000000016" height="24" rx="5"></rect><text fill="currentColor" x="224.19" y="70" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="254.4" y="54" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="254" y="90" text-anchor="start">hibernated</text></g></g> <text fill="currentColor" x="178.07999999999998" y="18" text-anchor="middle">a request wakes it</text></svg>

wakinghibernated

A Sprite hibernates when nothing uses it and wakes on the next request, so an idle sandbox costs little.

## Declare a Sprite

A Sprite is a class. Props describe the sandbox. The Effect is the program that runs on it.

```typescript
import * as Fly from "@alchemy.run/fly";
import * as Effect from "effect/Effect";

export default class Box extends Fly.Sprite<Box>()(
  "Box",
  { main: import.meta.url },
  Effect.gen(function* () {
    return {};
  }),
) {}
```

`main: import.meta.url` is the bundle entrypoint. Alchemy bundles this file with Rolldown, writes it onto the Sprite, and runs it as a Sprite service on `port` (default `3000`).

## Serve HTTP with fetch

Return `fetch` from the constructor Effect to boot an HTTP server. The Sprite URL proxies to `port`.

```typescript
import * as Fly from "@alchemy.run/fly";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Box extends Fly.Sprite<Box>()(
  "Box",
  { main: import.meta.url },
  Effect.gen(function* () {
    return {};
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello")),
    };
  }),
) {}
```

## The public URL

Yield the Sprite in the Stack. `box.url` is `https://{name}-….sprites.app`.

```typescript
export default Alchemy.Stack(
  "MyApp",
  { providers: Fly.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const box = yield* Box;
    return { url: box.url };
  }),
);
```

## URL auth

`urlAuth` is `public` or `sprite`. Default is `public` so `url` answers without extra auth.

```typescript
export default class Box extends Fly.Sprite<Box>()(
  "Box",
  { main: import.meta.url },
  { main: import.meta.url, urlAuth: "sprite" },
```

## Config

Yield `Config` in the constructor. Alchemy reads the value from the env of whoever deploys and writes it onto the Sprite. Do not pass `env: { ... }` on a Sprite.

Yield `FileSystem.FileSystem` in the constructor, never inside `fetch`.

```typescript
import * as Config from "effect/Config";
import * as FileSystem from "effect/FileSystem";
import * as Redacted from "effect/Redacted";

export default class Box extends Fly.Sprite<Box>()(
  "Box",
  { main: import.meta.url },
  Effect.gen(function* () {
    const apiKey = yield* Config.Redacted("API_KEY");
    const fs = yield* FileSystem.FileSystem;
    yield* fs.makeDirectory("/tmp", { recursive: true });

    return {
      fetch: Effect.gen(function* () {
        const token = Redacted.value(apiKey);
        return HttpServerResponse.text("ok");
      }),
    };
  }),
) {}
```

## Service vs Sprite

Reach for a [Service](services.md) when the process should stay up on a Fly Machine behind `{app}.fly.dev`. Reach for a Sprite when the sandbox can sleep. A Service needs Docker on the machine that deploys. A Sprite does not.

## Exec

[`Exec`](https://alchemy.run/providers/fly/reference/sprite#exec) runs a command on the Sprite. Provide [`ExecHttp`](https://alchemy.run/providers/fly/reference/sprite#exechttp).

```typescript
const exec = yield* Fly.Exec(Box);
const result = yield* exec({ cmd: ["ls", "-la"] });
```

## Checkpoint

[`Checkpoint`](https://alchemy.run/providers/fly/reference/sprite#checkpoint) snapshots and restores the Sprite disk. Provide [`CheckpointHttp`](https://alchemy.run/providers/fly/reference/sprite#checkpointhttp).

```typescript
const checkpoint = yield* Fly.Checkpoint(Box);
yield* checkpoint.create({ comment: "before" });
yield* checkpoint.restore("v1");
```

## A stable name

Omit `name` and Alchemy generates one from the stack, stage, and logical ID.

```typescript
export default class Box extends Fly.Sprite<Box>()(
  "Box",
  { main: import.meta.url, name: "box" },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello")),
    };
  }),
) {}
```

## Where next

[Services](services.md) are always-on Machines on an App. [Secrets](../data/secrets.md) covers `Config.Redacted`. The [`Sprite` reference](https://alchemy.run/providers/fly/reference/sprite#sprite) lists every prop.
