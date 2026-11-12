---
url: https://alchemy.run/archil/tutorial/part-2
title: "Part 2: Fork It for Each Request"
description: "A program in its own Archil sandbox that forks the set-up sandbox for each request, runs a command, and deletes the fork."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Bind CreateSandbox to the sandbox

```typescript
import * as Archil from "@alchemy.run/archil";
import * as Effect from "effect/Effect";
import { Golden } from "./Golden.ts";

export default class Api extends Archil.Sandbox<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const sandboxes = yield* Archil.CreateSandbox(Golden);
    return {};
  }).pipe(Effect.provide(Archil.CreateSandboxHttp)),
) {}
```

`Api` is a program Alchemy runs in an Archil sandbox of its own, at a public URL. Bound to `Golden`, every sandbox this client creates is a fork of it: its memory and disk, with the repository already cloned.

## Run a command in a fresh fork

```typescript
const sandboxes = yield* Archil.CreateSandbox(Golden);

const run = Effect.fn("run")(function* (command: string) {
  const id = (yield* Random.nextIntBetween(0, 2 ** 31)).toString(36);
  const sandbox = yield* Effect.acquireRelease(sandboxes.create({ name: \`run-${id}\` }), (sandbox) =>
    Effect.ignore(sandbox.delete()),
  );
  return yield* sandbox.exec(command, { cwd: "/work/is" });
});
```

A fork takes a fraction of a second whatever the disk’s size, and writes to it never reach `Golden`. `acquireRelease` deletes the fork when the request’s scope closes.

## Serve it

```typescript
return {};
return {
  fetch: Effect.gen(function* () {
    const request = yield* HttpServerRequest;
    const url = new URL(request.url, "http://api");
    if (request.method === "POST" && url.pathname === "/run") {
      return yield* HttpServerResponse.json(yield* run(yield* request.text).pipe(Effect.scoped));
    }
    return HttpServerResponse.text("not found", { status: 404 });
  }).pipe(Effect.orDie),
};
```

## Deploy it

```typescript
Effect.gen(function* () {
  const golden = yield* Golden;
  return { sandbox: golden.name };
  const api = yield* Api;
  return { sandbox: golden.name, url: api.url };
}),
```

```sh
pnpm alchemy deploy
curl -X POST "$URL/run" -d 'git log -1 --format=%s'
```

`Golden` is unchanged, so this deploy only adds the program.

## Next

[Part 3](part-3.md) gives each user a fork of their own.
