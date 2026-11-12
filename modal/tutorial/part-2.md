---
url: https://alchemy.run/modal/tutorial/part-2
title: "Part 2: Start Sandboxes from It"
description: "A Cloudflare Worker that starts a Modal sandbox from the warm image for each command, and reads back its output."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Bind sandboxes and volumes in a Worker

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { MyEnv, Warm } from "./Sandbox.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url, compatibility: { flags: ["nodejs_compat"] } },
  Effect.gen(function* () {
    const sandboxes = yield* Modal.CreateSandbox(Warm);
    const volumes = yield* Modal.CreateVolume(MyEnv);
    return {};
  }).pipe(Effect.provide(Layer.mergeAll(Modal.CreateSandboxHttp, Modal.CreateVolumeHttp))),
) {}
```

`CreateSandbox(Warm)` boots every sandbox from the warm image. `CreateVolume(MyEnv)` creates volumes by name in the environment.

## Run a command in a fresh sandbox

```typescript
const volumes = yield* Modal.CreateVolume(MyEnv);

const run = Effect.fn("run")(function* (user: string, command: string) {
  const home = yield* volumes.getOrCreate(\`home-${user}\`);
  const sandbox = yield* sandboxes.create({
    command: ["sh", "-c", \`cd /work && { ${command}; } > /home/user/.output 2>&1\`],
    mounts: [{ volume: home, path: "/home/user" }],
    timeout: "5 minutes",
  });
  const { exitCode } = yield* sandbox.wait("50 seconds");
  return { exitCode, output: yield* home.readFileString("/.output") };
});
```

The repository is already there: the sandbox boots from the image, so nothing clones at request time. The sandbox runs the command as its program and stops when it exits. A Worker can’t stream a sandbox’s output yet, so the command writes it to the user’s volume, mounted at `/home/user`, and the Worker reads it back.

## Serve it

```typescript
return {};
return {
  fetch: Effect.gen(function* () {
    const request = yield* HttpServerRequest;
    const url = new URL(request.url, "http://api");
    const user = /^\/users\/([a-z0-9-]+)\/run$/.exec(url.pathname)?.[1];
    if (request.method === "POST" && user !== undefined) {
      return yield* HttpServerResponse.json(yield* run(user, yield* request.text));
    }
    return HttpServerResponse.text("not found", { status: 404 });
  }).pipe(Effect.orDie),
};
```

## Deploy it

```typescript
Effect.gen(function* () {
  const warm = yield* Warm;
  return { image: warm.imageId };
  const api = yield* Api;
  return { url: api.url.as<string>(), image: warm.imageId };
}),
```

```sh
pnpm alchemy deploy
curl -X POST "$URL/users/ada/run" -d 'git log -1 --format=%s'
```

The images are unchanged, so this deploy only adds the Worker.

## Next

[Part 3](part-3.md) gives each user files of their own.
