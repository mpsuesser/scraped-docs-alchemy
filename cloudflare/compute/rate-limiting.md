---
url: https://alchemy.run/cloudflare/compute/rate-limiting
title: "Rate limiting"
description: "Throttle requests inside a Worker with Cloudflare's Rate Limiting binding — declare the limit on the binding, count arbitrary keys at runtime, handle failures as typed errors."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

The Rate Limiting binding gives a Worker a counter it can consult on every request. Pass a key, such as an IP, a user id, or an API token, and get back whether that key is within its budget. There is no backing cloud resource to deploy; the limit configuration lives on the binding itself and ships with the Worker.

## Bind a rate limit to a Worker

`yield*` `Cloudflare.RateLimit(name, props)` in the Worker’s Construction phase and provide `RateLimitBinding` as a layer:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default Cloudflare.Worker(
  "ThrottledWorker",
  { main: import.meta.url },
  Effect.gen(function* () {
    const throttle = yield* Cloudflare.RateLimit("THROTTLE", {
      namespaceId: 1001,
      simple: { limit: 10, period: 60 },
    });

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const ip = request.headers["cf-connecting-ip"] ?? "unknown";

        const { success } = yield* throttle
          .limit({ key: ip })
          .pipe(Effect.orDie);

        return success
          ? HttpServerResponse.text("ok")
          : HttpServerResponse.text("rate limited", { status: 429 });
      }),
    };
  }).pipe(Effect.provide(Cloudflare.Workers.RateLimitBinding)),
);
```

`simple.limit` is the number of requests allowed per `simple.period` seconds. The period must be `10` or `60`. `namespaceId` is a number or string that uniquely identifies this limit configuration within the account; reuse the same id to share one budget across Workers, use distinct ids for independent limits.

## Count independent keys

Every key passed to `limit` gets its own budget. With `limit: 2, period: 10`, two calls for `"alice"` succeed and the third is denied, while `"bob"` still has a full budget:

```typescript
const a1 = yield* throttle.limit({ key: "alice" }); // { success: true }
const a2 = yield* throttle.limit({ key: "alice" }); // { success: true }
const a3 = yield* throttle.limit({ key: "alice" }); // { success: false }
const b1 = yield* throttle.limit({ key: "bob" });   // { success: true }
```

<svg viewBox="0 0 684.8 124" style="max-width: 788px" role="img" aria-label="With limit 2 per 10 seconds, alice's third call is denied until the next period. bob has a budget of his own."><line stroke-opacity="0.2" stroke="currentColor" x1="448.8" y1="24" x2="448.8" y2="118"></line><g><text fill="currentColor" x="4" y="56">key: "alice"</text> <g><rect stroke="currentColor" fill="none" x="124.00000000000001" y="40" width="61.60000000000001" height="24" rx="5"></rect><text fill="currentColor" x="154.8" y="56" text-anchor="middle">ok</text></g> <g><rect stroke="currentColor" fill="none" x="196.8" y="40" width="61.599999999999966" height="24" rx="5"></rect><text fill="currentColor" x="227.6" y="56" text-anchor="middle">ok</text></g> <g><rect stroke="currentColor" fill="none" x="275.2" y="40" width="78.40000000000003" height="24" rx="5"></rect><text fill="currentColor" x="314.4" y="56" text-anchor="middle">denied</text></g> <g><rect stroke="currentColor" fill="none" x="465.6" y="40" width="61.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="496.40000000000003" y="56" text-anchor="middle">ok</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="684.8" y2="74"></line><text fill="currentColor" x="4" y="100">key: "bob"</text> <g><rect stroke="currentColor" fill="none" x="286.4" y="84" width="61.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="317.2" y="100" text-anchor="middle">ok</text></g></g> <text fill="currentColor" x="448.8" y="18" text-anchor="middle">next 10s period</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="With limit 2 per 10 seconds, alice's third call is denied until the next period. bob has a budget of his own."><line stroke-opacity="0.2" stroke="currentColor" x1="190.79999999999998" y1="24" x2="190.79999999999998" y2="138"></line><g><text fill="currentColor" x="0" y="44">key: "alice"</text> <g><rect stroke="currentColor" fill="none" x="6.36" y="54" width="34.980000000000004" height="24" rx="5"></rect><text fill="currentColor" x="23.85" y="70" text-anchor="middle">ok</text></g> <g><rect stroke="currentColor" fill="none" x="47.699999999999996" y="54" width="34.98000000000001" height="24" rx="5"></rect><text fill="currentColor" x="65.19" y="70" text-anchor="middle">ok</text></g> <g><rect stroke="currentColor" fill="none" x="92.22" y="54" width="44.52000000000001" height="24" rx="5"></rect><text fill="currentColor" x="142.74" y="70" text-anchor="start">denied</text></g> <g><rect stroke="currentColor" fill="none" x="200.34" y="54" width="34.97999999999999" height="24" rx="5"></rect><text fill="currentColor" x="217.82999999999998" y="70" text-anchor="middle">ok</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">key: "bob"</text> <g><rect stroke="currentColor" fill="none" x="98.58" y="108" width="34.980000000000004" height="24" rx="5"></rect><text fill="currentColor" x="116.07" y="124" text-anchor="middle">ok</text></g></g> <text fill="currentColor" x="190.79999999999998" y="18" text-anchor="middle">next 10s period</text></svg>

With limit 2 per 10 seconds, alice's third call is denied until the next period. bob has a budget of his own.

Counting is approximate and local to the Cloudflare location serving the request. The binding is built for cheap, fast abuse protection, not globally consistent quotas. For exact global counting, put the counter in a [Durable Object](durable-objects.md).

## Handle failures as typed errors

`limit` fails with a typed `RateLimitError` if the runtime call itself blows up (as opposed to the key being over budget, which is the `success: false` case). It lives in the Effect error channel like any other tagged error:

```typescript
const { success } = yield* throttle.limit({ key }).pipe(
  // fail open: allow the request if the limiter itself errors
  Effect.catchTag("RateLimitError", () =>
    Effect.succeed({ success: true }),
  ),
);
```

## Declare the binding as env metadata

Workers whose `main` module exports a plain async `fetch` handler declare the limit on `env` instead. `InferEnv` types it as Cloudflare’s native `RateLimit` binding:

```typescript
export const Worker = Cloudflare.Worker("Worker", {
  main: "./src/worker.ts",
  env: {
    THROTTLE: Cloudflare.RateLimit("THROTTLE", {
      namespaceId: 1001,
      simple: { limit: 10, period: 60 },
    }),
  },
});

export type WorkerEnv = Cloudflare.InferEnv<typeof Worker>;
```

```typescript
import type { WorkerEnv } from "../alchemy.run.ts";

export default {
  async fetch(req: Request, env: WorkerEnv) {
    const { success } = await env.THROTTLE.limit({
      key: req.headers.get("cf-connecting-ip") ?? "unknown",
    });
    return new Response(success ? "ok" : "rate limited", {
      status: success ? 200 : 429,
    });
  },
};
```

## Where next

- [Workers](workers.md) — the two-phase Worker model the binding lives inside.
- [Durable Objects](durable-objects.md) — exact, globally consistent counters when approximate isn’t enough.
- [RateLimit API reference](https://alchemy.run/providers/cloudflare/workers#ratelimit) — props and client types.
