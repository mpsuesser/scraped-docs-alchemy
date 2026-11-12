---
url: https://alchemy.run/modal/services/functions
title: "Functions"
description: "Run your program as a Modal Function that other runtimes call, on a schedule, or on the web."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A `Modal.Function` runs your program in containers Modal scales with demand, down to zero when idle. It belongs to a `Modal.App`, and other runtimes call it, a schedule runs it, or the web reaches it.

## Modal starts containers as calls arrive

Each call is an **input**. When inputs arrive faster than one container serves them, Modal starts another, and stops each once it’s been idle for the scale-down window:

<svg viewBox="0 0 677.4 124" style="max-width: 779px" role="img" aria-label="The first call starts a container, and a second starts when calls pile up. Each stops after the scale-down window, and with no calls nothing runs or is billed."><line stroke-opacity="0.2" stroke="currentColor" x1="273.4" y1="24" x2="273.4" y2="118"></line><g><text fill="currentColor" x="4" y="56">container 1</text> <g><rect stroke="currentColor" fill="none" x="127.80000000000001" y="40" width="56" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="183.8" y="40" width="100.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="234.20000000000002" y="56" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="295.79999999999995" y="40" width="100.80000000000007" height="24" rx="5"></rect><text fill="currentColor" x="346.2" y="56" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="396.6" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="452.6" y="56" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="677.4" y2="74"></line><text fill="currentColor" x="4" y="100">container 2</text> <g><rect stroke="currentColor" fill="none" x="273.4" y="84" width="56" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="329.4" y="84" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="379.79999999999995" y="100" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="430.19999999999993" y="84" width="112.00000000000011" height="24" rx="5"></rect><text fill="currentColor" x="486.2" y="100" text-anchor="middle">idle</text></g></g> <text fill="currentColor" x="273.4" y="18" text-anchor="middle">calls pile up</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="The first call starts a container, and a second starts when calls pile up. Each stops after the scale-down window, and with no calls nothing runs or is billed."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="24" x2="95.39999999999999" y2="138"></line><g><text fill="currentColor" x="0" y="44">container 1</text> <g><rect stroke="currentColor" fill="none" x="12.72" y="54" width="31.799999999999997" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="44.519999999999996" y="54" width="57.24000000000001" height="24" rx="5"></rect><text fill="currentColor" x="73.14" y="70" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="108.11999999999999" y="54" width="57.24000000000002" height="24" rx="5"></rect><text fill="currentColor" x="136.74" y="70" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="165.36" y="54" width="63.599999999999966" height="24" rx="5"></rect><text fill="currentColor" x="197.16" y="70" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">container 2</text> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="108" width="31.80000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="127.2" y="108" width="57.239999999999995" height="24" rx="5"></rect><text fill="currentColor" x="155.82" y="124" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="184.44" y="108" width="63.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="216.24" y="124" text-anchor="middle">idle</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">calls pile up</text></svg>

callcold startidle

The first call starts a container, and a second starts when calls pile up. Each stops after the scale-down window, and with no calls nothing runs or is billed.

## Group it in an App

```typescript
import * as Modal from "@alchemy.run/modal";

export const Inference = Modal.App("Inference", {});
```

## Write the function

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import { Inference } from "./Inference.ts";

export default class Embed extends Modal.Function<Embed>()(
  "Embed",
  { main: import.meta.url, app: Inference },
  Effect.gen(function* () {
    return {
      embed: Effect.fn(function* (text: string) {
        return { tokens: text.split(/\s+/).length };
      }),
    };
  }),
) {}
```

Every function the program returns is an RPC method. Alchemy deploys each one as a method of the Modal function, so Modal queues, retries and autoscales its calls.

## Call it from a Worker

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import Embed from "./Embed.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url, compatibility: { flags: ["nodejs_compat"] } },
  Effect.gen(function* () {
    const embed = yield* Modal.InvokeFunction(Embed);
    return {
      fetch: Effect.gen(function* () {
        const { tokens } = yield* embed.embed("hello modal");
        return yield* HttpServerResponse.json({ tokens });
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(Modal.InvokeFunctionHttp)),
) {}
```

The client is typed from `Embed` ’s methods. The binding also gives the Worker its own key, which the function checks on every call, so only bound callers can call its methods. Arguments and results travel as JSON, and a typed failure reaches the caller by its `_tag`, as with any [schemaless RPC](../../apis/schemaless.md).

## Call it from another Function

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import Embed from "./Embed.ts";
import { Inference } from "./Inference.ts";

export default class Relay extends Modal.Function<Relay>()(
  "Relay",
  { main: import.meta.url, app: Inference },
  Effect.gen(function* () {
    const embed = yield* Modal.InvokeFunction(Embed);
    return {
      relay: Effect.fn(function* (text: string) {
        return yield* embed.embed(text);
      }),
    };
  }).pipe(Effect.provide(Modal.InvokeFunctionHttp)),
) {}
```

The binding is the same one a Worker uses. Inside Modal, `Relay` calls Modal’s API as its own container, so no Modal token is stored in it, but its calls still carry the key its binding issued.

## Callers without a binding reach only fetch

RPC methods answer only callers whose binding issued a key. A script, `modal run`, or Python’s `modal.Cls.from_name` gets `401`. To call a function from those, give it no RPC methods: a function that serves only `fetch` takes [arguments](#call-it-with-arguments) from any caller Modal lets through.

## Return a stream

```typescript
return {
  embed: Effect.fn(function* (text: string) {
    return { tokens: text.split(/\s+/).length };
  }),
  chunks: (text: string) => Stream.fromIterable(text.split(/\s+/)),
};
```

A method can return a `Stream`. The call collects it in the container and the caller reads it once the call finishes. To stream as values are produced, serve the method from a [Server](servers.md).

## Run on a schedule

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Inference } from "./Inference.ts";

export default class Digest extends Modal.Function<Digest>()(
  "Digest",
  { main: import.meta.url, app: Inference, schedule: { cron: "0 9 * * *", timezone: "Europe/London" } },
  Effect.gen(function* () {
    return {
      fetch: Effect.gen(function* () {
        yield* Effect.log("sending the digest");
        return yield* HttpServerResponse.json(null);
      }),
    };
  }),
) {}
```

`schedule` also takes `{ period: "1 hour" }`. Each tick reaches `fetch` as `POST /__modal/schedule`. A function serves calls to its methods, a web endpoint or a schedule, so a scheduled function has no RPC methods.

## Write it without Effect

```typescript
import * as Modal from "@alchemy.run/modal";
import { Inference } from "./Inference.ts";

export const Embed = Modal.Function("Embed", {
  main: "./src/embed-server.ts",
  app: Inference,
  env: { MODEL: "small" },
});
export type EmbedEnv = Modal.InferEnv<typeof Embed>;
```

```typescript
import type { EmbedEnv } from "./Embed.ts";

const env = process.env as unknown as EmbedEnv;

Bun.serve({
  port: Number(process.env.PORT),
  async fetch(request) {
    const { args } = (await request.json()) as { args: [string] };
    return Response.json({ model: env.MODEL, tokens: args[0].split(/\s+/).length });
  },
});
```

The program runs on Bun. Calls arrive as `POST /__modal/call` with `{ args, kwargs }`.

## Call it with arguments

```typescript
const embed = yield* Modal.InvokeFunction(Embed);
const result = yield* embed.call<{ tokens: number }>(["hello modal"], { model: "small" });
```

A function without RPC methods, like this one, takes positional and keyword arguments as CBOR, as Python functions do.

## Fan out over many inputs

```typescript
const results = yield* embed.map<{ tokens: number }>([["a"], ["b c"], ["d e f"]]);
```

Modal spreads the inputs across containers and returns the results in order.

## Spawn a call and read it later

```typescript
const job = yield* embed.spawn(["a long document"]);
const result = yield* embed.result(job, { timeout: "10 minutes" });
```

## Add GPUs, volumes and warm containers

```typescript
export default class Generate extends Modal.Function<Generate>()(
  "Generate",
  {
    main: import.meta.url,
    app: Inference,
    gpu: "L4",
    mounts: [{ volume: Weights, path: "/models" }],
    minContainers: 1,
    maxConcurrentInputs: 8,
    timeout: "10 minutes",
  },
  program,
) {}
```

## Modal starts it through Python

Modal starts every container with its Python entrypoint. Alchemy adds standalone Python to your image and a small entrypoint module, which starts your program and forwards calls to it over localhost. Set `python: false` if your base image already has Python.
