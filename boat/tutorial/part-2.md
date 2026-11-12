---
url: https://alchemy.run/boat/tutorial/part-2
title: "Part 2: Start Sandboxes from It"
description: "A Cloudflare Worker that starts a fresh Boat sandbox from the Golden snapshot for each request."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Bind CreateSandbox to the snapshot

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { Golden } from "./Golden.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const sandboxes = yield* Boat.CreateSandbox(Golden);
    return {};
  }).pipe(Effect.provide(Boat.CreateSandboxHttp)),
) {}
```

Every sandbox this client creates starts from `Golden`. At deploy, the Worker gets a Boat key of its own that creates sandboxes, works in them, and deletes only the ones it created.

## Run a command in a fresh sandbox

```typescript
Effect.gen(function* () {
  const sandboxes = yield* Boat.CreateSandbox(Golden);

  const run = Effect.fn("run")(function* (command: string) {
    const sandbox = yield* Effect.acquireRelease(
      sandboxes.create({ noEnv: true, autoStop: "5 minutes" }),
      (sandbox) => Effect.ignore(sandbox.delete()),
    );
    return yield* sandbox.exec(command, { cwd: "/home/user/repo", timeout: "1 minute" });
  });
```

The repository is already there: the sandbox starts from the snapshot’s disk, so nothing clones at request time. `acquireRelease` deletes the sandbox when the request’s scope closes, whatever happens.

## Serve it

```typescript
return {};
return {
  fetch: Effect.gen(function* () {
    const request = yield* HttpServerRequest;
    const url = new URL(request.url, "http://api");
    if (request.method === "POST" && url.pathname === "/run") {
      const result = yield* run(yield* request.text).pipe(Effect.scoped);
      return yield* HttpServerResponse.json(result);
    }
    return HttpServerResponse.text("not found", { status: 404 });
  }).pipe(Effect.orDie),
};
```

## Deploy it

```typescript
Effect.gen(function* () {
  const golden = yield* Golden;
  return { snapshot: golden.name };
  const api = yield* Api;
  return { snapshot: golden.name, url: api.url.as<string>() };
}),
```

```sh
pnpm alchemy deploy
```

`Golden` is unchanged, so this deploy only adds the Worker. Run a command in the repository:

```sh
curl -X POST "$URL/run" -d 'git log -1 --format=%s'
```

## Next

[Part 3](part-3.md) gives each user a sandbox of their own.
