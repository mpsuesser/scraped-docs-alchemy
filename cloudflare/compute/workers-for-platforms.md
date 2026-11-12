---
url: https://alchemy.run/cloudflare/compute/workers-for-platforms
title: "Workers for Platforms"
description: "Run your customers' Workers in your own account — create a dispatch namespace, upload user Workers into it, and route requests from a platform Worker with the Get binding."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Workers for Platforms lets you run your customers’ code on Cloudflare’s edge inside your own account. Customer (“user”) Workers live in a **dispatch namespace** and get no routes or URLs of their own. Instead, a **platform Worker** you control receives every request and forwards it to the right user Worker by script name.

## Create a dispatch namespace

Declare the namespace as a shared constant so both the Stack and the platform Worker can reference the same resource:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Customers = Cloudflare.WorkersForPlatforms.DispatchNamespace(
  "Customers",
  { name: "my-platform-customers" },
);
```

The `name` is the namespace’s identity. There is no rename API, so changing it triggers a replacement (omit it to get a generated name). Deleting a namespace also deletes every script uploaded into it.

## Upload user Workers into the namespace

Setting a Worker’s `namespace` prop deploys it *into* the namespace as a user Worker instead of as a routable account-level script:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { Customers } from "./src/namespace.ts";
import PlatformWorker from "./src/platform.ts";

export default Alchemy.Stack(
  "Platform",
  { providers: Cloudflare.providers(), state: Cloudflare.state() },
  Effect.gen(function* () {
    const namespace = yield* Customers;

    yield* Cloudflare.Worker("CustomerA", {
      namespace: namespace.name,
      script: \`export default {
        async fetch(request) {
          return Response.json({ handledBy: "customer-a" });
        },
      };\`,
    });

    const platform = yield* PlatformWorker; // defined next
    return { url: platform.url.as<string>() };
  }),
);
```

Referencing the namespace’s `name` output establishes the dependency edge, so the namespace is created before the user Worker is uploaded into it (and the script is deleted first on destroy). The `script` form takes raw ESM source, typically whatever code your customer uploaded. `main` with a local entrypoint works too.

## Dispatch from the platform Worker

`Cloudflare.WorkersForPlatforms.Get(namespace)` binds the namespace to the Worker and returns a dynamic-dispatch client; `dispatch.get(name)` looks up a user Worker by script name and returns a `Fetcher`:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Customers } from "./namespace.ts";

export default class PlatformWorker extends Cloudflare.Worker<PlatformWorker>()(
  "PlatformWorker",
  { main: import.meta.filename, workersDev: true },
  Effect.gen(function* () {
    const dispatch = yield* Cloudflare.WorkersForPlatforms.Get(Customers);

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        // Route however your platform maps requests to customers —
        // here the first path segment is the script name.
        const url = new URL(request.url, "http://platform");
        const match = url.pathname.match(/^\/dispatch\/([^/]+)(\/.*)?$/);
        if (!match) {
          return HttpServerResponse.text("platform ok");
        }
        const [, scriptName, rest] = match;
        const userWorker = yield* dispatch.get(scriptName).pipe(Effect.orDie);
        const response = yield* Effect.promise(() =>
          userWorker.fetch(new Request(\`https://user-worker${rest ?? "/"}\`)),
        );
        return HttpServerResponse.fromWeb(response);
      }),
    };
  }).pipe(Effect.provide(Cloudflare.WorkersForPlatforms.GetBinding)),
) {}
```

<svg viewBox="0 0 704 176" style="max-width: 810px" role="img" aria-label="User Workers in the Customers namespace have no URL. Every request reaches the platform Worker, which picks one by script name and forwards the rest of the path."><defs><marker id="alc-fan-arrow-7le2hl" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="283" y="8" width="217" height="160" rx="10"></rect><text fill="currentColor" x="293" y="26">Platform · Worker</text></g> <rect stroke="currentColor" fill="none" x="8" y="63.5" width="223" height="49" rx="8"></rect><text fill="currentColor" x="20" y="83.5">request</text> <text fill="currentColor" x="20" y="103.5" xml:space="preserve">GET /dispatch/customer-a/hello</text> <g><path stroke="currentColor" fill="none" d="M 231 88 C 262 88, 262 64, 293 64" marker-end="url(#alc-fan-arrow-7le2hl)"></path><g><rect stroke="currentColor" fill="none" x="295" y="38" width="193" height="52" rx="7"></rect><text fill="currentColor" x="307" y="55">dispatch</text> <text fill="currentColor" x="307" y="72.5" xml:space="preserve">dispatch.get("customer-a")</text> <line stroke-opacity="0.2" stroke="currentColor" x1="488" y1="64" x2="574" y2="64" marker-end="url(#alc-fan-arrow-7le2hl)"></line></g><rect stroke="currentColor" fill="none" x="576" y="41" width="120" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="636" y="61" text-anchor="middle" opacity="1">customer-a</text> <text fill="currentColor" x="636" y="77" text-anchor="middle">user Worker</text></g> <g><path stroke="currentColor" fill="none" d="M 231 88 C 262 88, 262 130, 293 130"></path><g><rect stroke="currentColor" fill="none" x="295" y="104" width="193" height="52" rx="7"></rect><text fill="currentColor" x="307" y="121">dispatch</text> <text fill="currentColor" x="307" y="138.5" xml:space="preserve">dispatch.get("customer-b")</text> <text fill="currentColor" x="500" y="134">no match</text></g> <rect stroke="currentColor" fill="none" x="576" y="107" width="120" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="636" y="127" text-anchor="middle" opacity="0.6">customer-b</text> <text fill="currentColor" x="636" y="143" text-anchor="middle">user Worker</text></g></svg> <svg viewBox="0 0 318 385" style="max-width: 366px" role="img" aria-label="User Workers in the Customers namespace have no URL. Every request reaches the platform Worker, which picks one by script name and forwards the rest of the path."><defs><marker id="alc-fan-arrow-n-7le2hl" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="14" y="63" width="304" height="320" rx="10"></rect><text fill="currentColor" x="24" y="81">Platform · Worker</text></g> <rect stroke="currentColor" fill="none" x="0" y="0" width="223" height="49" rx="8"></rect><text fill="currentColor" x="12" y="20">request</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">GET /dispatch/customer-a/hello</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="49" x2="8" y2="273"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="127" x2="20" y2="127" marker-end="url(#alc-fan-arrow-n-7le2hl)"></line><g><rect stroke="currentColor" fill="none" x="22" y="101" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="118">dispatch</text> <text fill="currentColor" x="34" y="135.5" xml:space="preserve">dispatch.get("customer-a")</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="153" x2="166" y2="181" marker-end="url(#alc-fan-arrow-n-7le2hl)"></line></g><rect stroke="currentColor" fill="none" x="106" y="183" width="120" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="203" text-anchor="middle" opacity="1">customer-a</text> <text fill="currentColor" x="166" y="219" text-anchor="middle">user Worker</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="273" x2="20" y2="273"></line><g><rect stroke="currentColor" fill="none" x="22" y="247" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="264">dispatch</text> <text fill="currentColor" x="34" y="281.5" xml:space="preserve">dispatch.get("customer-b")</text> <text fill="currentColor" x="166" y="318" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="106" y="329" width="120" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="349" text-anchor="middle" opacity="0.6">customer-b</text> <text fill="currentColor" x="166" y="365" text-anchor="middle">user Worker</text></g></svg>

User Workers in the Customers namespace have no URL. Every request reaches the platform Worker, which picks one by script name and forwards the rest of the path.

Provide `GetBinding` as a layer so the client resolves the native `dispatch_namespace` runtime binding. A request to `/dispatch/customer-a/hello` is forwarded to the `customer-a` user Worker with the trailing path intact; dispatching to a script name that doesn’t exist in the namespace surfaces as an error from the edge, never a user-Worker response.

## Or bind on env

For a plain async handler, pass the namespace on the Worker’s `env` instead. `InferEnv` types it as the native runtime binding:

```typescript
const platform = Cloudflare.Worker("Platform", {
  main: "./src/handler.ts",
  workersDev: true,
  env: { DISPATCH: Customers },
});
type Env = Cloudflare.InferEnv<typeof platform>;

// src/handler.ts
export default {
  async fetch(request: Request, env: Env) {
    return env.DISPATCH.get("customer-a").fetch(request);
  },
};
```

`env.DISPATCH.get(name)` returns the same `Fetcher` the Effect-native client wraps. Pick whichever style matches the rest of the Worker.

## Where next

- [Workers](workers.md) — the two-phase Worker model the platform Worker is built on.
- [DispatchNamespace API reference](https://alchemy.run/providers/cloudflare/workersforplatforms#dispatchnamespace)
- [Get API reference](https://alchemy.run/providers/cloudflare/workersforplatforms#get)
