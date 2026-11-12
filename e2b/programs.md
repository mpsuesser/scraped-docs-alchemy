---
url: https://alchemy.run/e2b/programs
title: "Template programs"
description: "Run your own Effect program, or a plain JavaScript program, in every sandbox."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Give a template a `main`, and every sandbox from it boots with your program already serving HTTP. Alchemy bundles it, copies it into the template, and E2B snapshots the template once the program answers. The program shares its microVM with whatever else runs there, so anything it can read, the sandbox’s other code can read too.

## The program shares the sandbox

user-42sandbox · microVM

ProgramBun · /app ·:3000

envconfig · binding values

agent codeexec · files

The program, its config and binding values, and any code you run in the sandbox all sit in the same microVM.

## Write an Effect program

```typescript
import * as E2B from "@alchemy.run/e2b";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Program extends E2B.Template<Program>()(
  "Program",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello from a sandbox")),
    };
  }),
) {}
```

The program runs with Bun from `/app`, on `oven/bun:1` by default. Set `image` or `dockerfile` for your own environment; it needs Bun.

## Call it from a Worker

```typescript
const sandboxes = yield* E2B.CreateSandbox(Program);
const sandbox = yield* sandboxes.create({ timeout: 300 });
const port = yield* sandbox.getTcpPort(3000);
const res = yield* port.fetch(HttpClientRequest.get("/"));
```

## Serve it from one sandbox

```typescript
import * as E2B from "@alchemy.run/e2b";
import * as Alchemy from "alchemy";
import * as Output from "alchemy/Output";
import * as Effect from "effect/Effect";
import Program from "./src/Program.ts";

export default Alchemy.Stack(
  "Program",
  { providers: E2B.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const sandbox = yield* E2B.Sandbox("ProgramSandbox", {
      template: Program,
      network: { allowPublicTraffic: true },
    });
    return { url: Output.interpolate\`https://3000-${sandbox.sandboxId}.${sandbox.domain}\` };
  }),
);
```

An `E2B.Sandbox` gives the program a stable URL. The sandbox pauses when it is idle and the next request resumes it, memory included. A new build of the program replaces the sandbox.

## Use bindings from the program

```typescript
export default class Program extends E2B.Template<Program>()(
  "Program",
  { main: import.meta.url },
  Effect.gen(function* () {
    const reports = yield* E2B.ReadVolume(Reports);
    return {
      fetch: Effect.gen(function* () {
        const report = yield* reports.readText("/latest.json");
        return HttpServerResponse.text(report);
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(E2B.ReadVolumeHttp)),
) {}
```

Any binding works in a template program, from E2B or another provider.

## Write a plain JavaScript program

```typescript
export const Server = E2B.Template("Server", {
  main: "./src/server.mjs",
  env: { GREETING: "hi" },
});
export type ServerEnv = E2B.InferEnv<typeof Server>;
```

```javascript
import { createServer } from "node:http";

createServer((_request, response) => {
  response.end(\`${process.env.GREETING} from a sandbox\`);
}).listen(Number(process.env.PORT ?? 3000), "0.0.0.0");
```

A `main` without an Effect is bundled and run as-is. It must listen on `PORT`.

## A deploy rebuilds the template

A deploy rebuilds the template when the program’s bundle changes. Sandboxes already running keep the old build, except an `E2B.Sandbox` declared from the template, which is replaced by one on the new build.
