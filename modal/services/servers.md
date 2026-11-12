---
url: https://alchemy.run/modal/services/servers
title: "Servers"
description: "Run a long-running HTTP server on Modal with regional routing and concurrency-based autoscaling."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A `Modal.Server` runs your program as an HTTP server. Requests are routed through one region, and Modal scales containers on concurrent requests.

## Each container serves many requests at once

A Function’s container takes one call at a time unless you raise it. A Server’s container serves requests side by side, and Modal adds a container when they pass its target:

<svg viewBox="0 0 677.4 184" style="max-width: 779px" role="img" aria-label="Container 1 serves three requests at once. When concurrency passes the target, Modal starts container 2."><line stroke-opacity="0.2" stroke="currentColor" x1="273.4" y1="24" x2="273.4" y2="178"></line><g><rect stroke="currentColor" fill="none" x="97.4" y="33" width="576" height="98" rx="8"></rect><text fill="currentColor" x="4" y="86">container 1</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="40" width="223.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="217.39999999999998" y="56" text-anchor="middle">GET /</text></g> <g><rect stroke="currentColor" fill="none" x="161.4" y="70" width="223.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="273.4" y="86" text-anchor="middle">GET /a</text></g> <g><rect stroke="currentColor" fill="none" x="217.4" y="100" width="223.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="329.4" y="116" text-anchor="middle">GET /b</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="134" x2="677.4" y2="134"></line><text fill="currentColor" x="4" y="160">container 2</text> <g><rect stroke="currentColor" fill="none" x="273.4" y="144" width="44.80000000000007" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="318.20000000000005" y="144" width="235.19999999999993" height="24" rx="5"></rect><text fill="currentColor" x="435.8" y="160" text-anchor="middle">GET /c</text></g></g> <text fill="currentColor" x="273.4" y="18" text-anchor="middle">past the target</text></svg> <svg viewBox="0 0 330 204" style="max-width: 379px" role="img" aria-label="Container 1 serves three requests at once. When concurrency passes the target, Modal starts container 2."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="24" x2="95.39999999999999" y2="198"></line><g><rect stroke="currentColor" fill="none" x="0" y="51" width="326" height="90" rx="8"></rect><text fill="currentColor" x="0" y="44">container 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="127.2" height="24" rx="5"></rect><text fill="currentColor" x="63.6" y="70" text-anchor="middle">GET /</text></g> <g><rect stroke="currentColor" fill="none" x="31.8" y="84" width="127.2" height="24" rx="5"></rect><text fill="currentColor" x="95.4" y="100" text-anchor="middle">GET /a</text></g> <g><rect stroke="currentColor" fill="none" x="63.6" y="114" width="127.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="127.19999999999999" y="130" text-anchor="middle">GET /b</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="144" x2="330" y2="144"></line><text fill="currentColor" x="0" y="158">container 2</text> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="168" width="25.440000000000012" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="120.84" y="168" width="133.56" height="24" rx="5"></rect><text fill="currentColor" x="187.62" y="184" text-anchor="middle">GET /c</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">past the target</text></svg>

requestcold start

Container 1 serves three requests at once. When concurrency passes the target, Modal starts container 2.

## Declare the Server

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Backend } from "./Backend.ts";

export default class Api extends Modal.Server<Api>()(
  "Api",
  { main: import.meta.url, app: Backend, routingRegion: "us-east" },
  Effect.gen(function* () {
    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }),
) {}
```

## Call it from a Worker

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as HttpClientRequest from "effect/http/HttpClientRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import Api from "./Api.ts";

export default class Gateway extends Cloudflare.Worker<Gateway>()(
  "Gateway",
  { main: import.meta.url },
  Effect.gen(function* () {
    const api = yield* Modal.FetchServer(Api);
    return {
      fetch: Effect.gen(function* () {
        const response = yield* api.fetch(HttpClientRequest.get("/"));
        return HttpServerResponse.text(yield* response.text);
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(Modal.FetchServerHttp)),
) {}
```

Servers require Modal’s proxy auth. The binding creates a `Modal.WebhookToken` for the Worker and sends its headers with every request.

## Add RPC methods

```typescript
Effect.gen(function* () {
  return {
    greet: Effect.fn(function* (name: string) {
      return \`hello ${name}\`;
    }),
    fetch: Effect.succeed(HttpServerResponse.text("ok")),
  };
}),
```

Every function the program returns besides `fetch` is an RPC method, served at `POST /__rpc__/<method>` next to your routes.

## Call a method

```typescript
const api = yield* Modal.FetchServer(Api);
const greeting = yield* api.greet("modal");
```

The client is typed from `Api` ’s methods and goes through Modal’s proxy auth like `fetch`. The binding also gives the Worker its own key, which the Server checks on every method call: callers without a binding get `401`. A method that returns a `Stream` streams to the caller as values are produced. See [schemaless RPC](../../apis/schemaless.md) for what methods can take and return.

## Call it from a Function

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import Api from "./Api.ts";
import { Backend } from "./Backend.ts";

export default class Relay extends Modal.Function<Relay>()(
  "Relay",
  { main: import.meta.url, app: Backend },
  Effect.gen(function* () {
    const api = yield* Modal.FetchServer(Api);
    return {
      relay: Effect.fn(function* (name: string) {
        return yield* api.greet(name);
      }),
    };
  }).pipe(Effect.provide(Modal.FetchServerHttp)),
) {}
```

The binding is the same one a Worker uses. A Server sits behind proxy auth even for programs inside Modal, so `Relay` still gets its own `Modal.WebhookToken`, and its method calls carry the key its binding issued.

## Callers without a binding reach only fetch

Any caller with a proxy auth token Modal accepts for the Server can reach the Server’s `fetch` routes. Only callers whose binding issued a key can call its methods: everyone else gets `401`.

## Serve the public without a token

```typescript
{ main: import.meta.url, app: Backend, unauthenticated: true }
```

An unauthenticated Server serves only `fetch`. Its RPC methods answer `404`, so the public can’t call them.

## Scale on concurrent requests

```typescript
{
  main: import.meta.url,
  app: Backend,
  minContainers: 2,
  targetConcurrency: 20,
  maxConcurrency: 50,
  exitGracePeriod: "30 seconds",
}
```

Each container aims for 20 requests at once and turns away more than 50 with `429`.

## Write it without Effect

```typescript
import * as Modal from "@alchemy.run/modal";
import { Backend } from "./Backend.ts";

export const Api = Modal.Server("Api", {
  main: "./src/api-server.ts",
  app: Backend,
  unauthenticated: true,
});
```

```typescript
Bun.serve({ port: Number(process.env.PORT), fetch: () => new Response("ok") });
```
