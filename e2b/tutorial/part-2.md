---
url: https://alchemy.run/e2b/tutorial/part-2
title: "Part 2: Start Sandboxes from It"
description: "A Cloudflare Worker that starts a fresh E2B sandbox from the Warm snapshot for each request."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Bind CreateSandbox to the snapshot

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as E2B from "@alchemy.run/e2b";
import * as Effect from "effect/Effect";
import { Warm } from "./Sandbox.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const sandboxes = yield* E2B.CreateSandbox(Warm);
    return {};
  }).pipe(Effect.provide(E2B.CreateSandboxHttp)),
) {}
```

Every sandbox this client creates boots from `Warm`. At deploy, the Worker gets an E2B API key of its own, as a secret.

## Run a command in a fresh sandbox

```typescript
const sandboxes = yield* E2B.CreateSandbox(Warm);

const run = Effect.fn("run")(function* (command: string) {
  const sandbox = yield* Effect.acquireRelease(sandboxes.create({ timeout: 120 }), (sandbox) =>
    Effect.ignore(sandbox.kill()),
  );
  return yield* sandbox.exec(command, { cwd: "/home/user/repo" });
});
```

The repository is already there: the sandbox resumes the snapshot, so nothing clones at request time. The sandbox is killed when the request ends, and `exec` returns `stdout`, `stderr` and `exitCode`.

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
  const warm = yield* Warm;
  return { snapshot: warm.templateId };
  const api = yield* Api;
  return { url: api.url.as<string>(), snapshot: warm.templateId };
}),
```

```sh
pnpm alchemy deploy
curl -X POST "$URL/run" --data 'git log -1 --format=%s'
```

The template and snapshot are unchanged, so this deploy only adds the Worker.

## Next

[Part 3](part-3.md) gives each user a sandbox of their own.
