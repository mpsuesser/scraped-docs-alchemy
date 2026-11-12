---
url: https://alchemy.run/cloudflare/compute/code-updates
title: "Code updates"
description: "What happens to running Workers and Durable Objects when you deploy, how to choose when Durable Objects pick up new code, and how to write tests that see the code you just deployed."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

When you deploy, new requests to your Worker run the new code within seconds. A Durable Object that is already running is different: it keeps running the code it started with until it is safe to switch. Knowing when that happens tells you how to write code that survives a deploy and tests that see what you just shipped.

## A Worker starts fresh on every deploy

Each deploy runs in new isolates. The Construction phase runs again, and anything the old isolate held in memory is gone:

```typescript
export default Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    // Runs once per isolate, so again after every deploy
    const counters = yield* Counter;

    return {
      fetch: Effect.gen(function* () {
        return HttpServerResponse.text("Hello from my Worker!");
      }),
    };
  }),
);
```

The new version can take a few seconds to reach every location, so retry the first request after a deploy. See [Retry the first request](../../testing/testing-a-stack.md#retry-the-first-request).

## A Durable Object keeps the code it started with

Each object runs one copy of your code, the inner Effect, from the moment it starts:

```typescript
export default class Counter extends Cloudflare.DurableObject<Counter>()(
  "Counter",
  Effect.gen(function* () {
    const state = yield* Cloudflare.DurableObjectState;

    return Effect.gen(function* () {
      // Runs each time the object starts
      let count = (yield* state.storage.get<number>("count")) ?? 0;

      return {
        increment: () =>
          Effect.gen(function* () {
            count += 1;
            yield* state.storage.put("count", count);
            return count;
          }),
      };
    });
  }),
) {}
```

An object stops when it goes idle. It hibernates after about 10 seconds without work, or is evicted after 70 to 140 seconds if something keeps it from hibernating, such as a pending `setTimeout` or an open outbound connection. Its next request starts it again on the latest code. `count` is read back from storage, so it survives. Anything kept only in memory does not.

<svg viewBox="0 0 677.4 168" style="max-width: 779px" role="img" aria-label="New requests reach the new Worker within seconds. A busy object keeps its old code until it goes idle, and an idle one starts on the new code with its next request."><line stroke-opacity="0.2" stroke="currentColor" x1="273.4" y1="24" x2="273.4" y2="162"></line><line stroke-opacity="0.2" stroke="currentColor" x1="497.4" y1="24" x2="497.4" y2="162"></line><g><text fill="currentColor" x="4" y="50">Worker</text> <text fill="currentColor" x="4" y="64">isolates</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="40" width="167.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="189.39999999999998" y="56" text-anchor="middle">version 1</text></g> <g><rect stroke="currentColor" fill="none" x="273.4" y="40" width="392" height="24" rx="5"></rect><text fill="currentColor" x="469.4" y="56" text-anchor="middle">version 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="677.4" y2="74"></line><text fill="currentColor" x="4" y="94">DO "lobby"</text> <text fill="currentColor" x="4" y="108">busy</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="84" width="392" height="24" rx="5"></rect><text fill="currentColor" x="301.4" y="100" text-anchor="middle">version 1, still serving</text></g> <g><rect stroke="currentColor" fill="none" x="497.4" y="84" width="168" height="24" rx="5"></rect><text fill="currentColor" x="581.4" y="100" text-anchor="middle">version 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="677.4" y2="118"></line><text fill="currentColor" x="4" y="138">DO "room-7"</text> <text fill="currentColor" x="4" y="152">idle</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="128" width="100.79999999999998" height="24" rx="5"></rect><text fill="currentColor" x="155.8" y="144" text-anchor="middle">version 1</text></g> <g><rect stroke="currentColor" fill="none" x="206.2" y="128" width="190.40000000000003" height="24" rx="5"></rect><text fill="currentColor" x="301.4" y="144" text-anchor="middle">hibernating</text></g> <g><rect stroke="currentColor" fill="none" x="396.6" y="128" width="268.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="531" y="144" text-anchor="middle">version 2</text></g></g> <text fill="currentColor" x="273.4" y="18" text-anchor="middle">deploy</text> <text fill="currentColor" x="497.4" y="18" text-anchor="middle">lobby goes idle</text></svg> <svg viewBox="0 0 330 213" style="max-width: 379px" role="img" aria-label="New requests reach the new Worker within seconds. A busy object keeps its old code until it goes idle, and an idle one starts on the new code with its next request."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="24" x2="95.39999999999999" y2="207"></line><line stroke-opacity="0.2" stroke="currentColor" x1="222.6" y1="24" x2="222.6" y2="207"></line><g><text fill="currentColor" x="0" y="44">Worker <tspan fill="currentColor" dx="8">isolates</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="47.699999999999996" y="70" text-anchor="middle">version 1</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="54" width="222.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="206.7" y="70" text-anchor="middle">version 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">DO "lobby" <tspan fill="currentColor" dx="8">busy</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="108" width="222.6" height="24" rx="5"></rect><text fill="currentColor" x="111.3" y="124" text-anchor="middle">version 1, still serving</text></g> <g><rect stroke="currentColor" fill="none" x="222.6" y="108" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="270.3" y="124" text-anchor="middle">version 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">DO "room-7" <tspan fill="currentColor" dx="8">idle</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="162" width="57.239999999999995" height="24" rx="5"></rect><text fill="currentColor" x="0" y="198" text-anchor="start">version 1</text></g> <g><rect stroke="currentColor" fill="none" x="57.239999999999995" y="162" width="108.12000000000002" height="24" rx="5"></rect><text fill="currentColor" x="111.30000000000001" y="178" text-anchor="middle">hibernating</text></g> <g><rect stroke="currentColor" fill="none" x="165.36" y="162" width="152.64" height="24" rx="5"></rect><text fill="currentColor" x="241.68" y="178" text-anchor="middle">version 2</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">deploy</text> <text fill="currentColor" x="222.6" y="18" text-anchor="middle">lobby goes idle</text></svg>

New requests reach the new Worker within seconds. A busy object keeps its old code until it goes idle, and an idle one starts on the new code with its next request.

## Deploys wait for objects to go idle

By default an object that is busy during a deploy keeps running its current code for up to 5 minutes while it waits to hibernate. Then it resets onto the new code:

```typescript
{
  main: import.meta.url,
  // the default
  codeUpdateStrategy: { mode: "deferred", maxDelay: "5 minutes" },
}
```

Requests in progress get time to finish, and [hibernatable WebSockets](hibernatable-websockets.md) stay connected when the object picks up the new code after hibernating.

## Choose how long to wait

Set `maxDelay` to anything from 0 to 24 hours:

```typescript
{
  main: import.meta.url,
  codeUpdateStrategy: { mode: "deferred", maxDelay: "1 minute" },
}
```

Waiting is best effort. An object can still reset sooner, for example when Cloudflare updates the Workers runtime.

## Switch objects to new code right away

Use `immediate` when every object must run the new code as soon as the deploy finishes, such as when you ship a fix during an incident:

```typescript
{
  main: import.meta.url,
  codeUpdateStrategy: { mode: "immediate" },
}
```

Every running object resets. A request it was handling fails with `Durable Object reset because its code was updated`. Data in `state.storage` is kept; in-memory values are lost.

## The strategy applies to each deploy

The strategy is not a setting stored on the Worker. It is sent with each deploy. Changing only `codeUpdateStrategy` does not deploy anything, because a redeploy would reset the objects it is meant to protect. Your next deploy uses the new value.

## Old and new code run side by side

While an object waits, a Worker running the new code can call an object still running the old code. Calling a method the old code doesn’t have fails:

```typescript
const counter = counters.getByName(name);
const next = yield* counter.increment();
const next = yield* counter.incrementBy(5); // missing on objects still running the old code
```

Ship the new method in one deploy and start calling it in the next, once `maxDelay` has passed. Keep method arguments and return values readable by both versions in the meantime.

## Write state as you go

An object can reset at any time: on deploy, when `maxDelay` runs out, or during a runtime update. Save each change to storage when it happens, as `increment` does, rather than when the object stops. Make operations safe to retry, so a caller can repeat a request that failed during a reset.

## Gradual deployments assign objects to versions

During a [gradual deployment](gradual-deployments.md), Cloudflare assigns each object to one version. Raising the new version’s traffic moves more objects onto it. Only objects that move wait for `maxDelay`; the rest keep running.

## Tests can see the old code

A test that deploys and then calls a Durable Object can reach an object an earlier deploy started. If that object stayed busy, it can answer with the old code for up to 5 minutes.

Deploy your test stages with `immediate`, so every object runs the code the test just deployed:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import { Stack } from "alchemy/Stack";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import Counter from "./counter.ts";

export default Cloudflare.Worker(
  "Worker",
  Stack.useSync(({ stage }) => ({
    main: import.meta.url,
    codeUpdateStrategy: stage === "prod" ? undefined : { mode: "immediate" },
  })),
  Effect.gen(function* () {
    const counters = yield* Counter;

    return {
      fetch: Effect.gen(function* () {
        return HttpServerResponse.text("Hello from my Worker!");
      }),
    };
  }),
);
```

Or give each test its own object. A new object always starts on the latest code:

```typescript
test(
  "increment starts at 1",
  Effect.gen(function* () {
    const { url } = yield* stack;
    const name = crypto.randomUUID();

    const response = yield* HttpClient.post(\`${url}/counter/${name}\`);
    expect(yield* response.text).toBe("1");
  }),
);
```

## Where next

- [Durable Objects](durable-objects.md) — define an object, persist its state, and call it from a Worker.
- [Accept WebSockets](hibernatable-websockets.md) — connections that stay open through hibernation and code updates.
- [Gradual deployments](gradual-deployments.md) — roll a deploy out to a share of traffic.
- [Testing a Stack](../../testing/testing-a-stack.md) — deploy once per suite and drive the live URL.
- [Cloudflare’s code updates docs](https://developers.cloudflare.com/durable-objects/deployments/durable-objects-code-updates/) — the platform feature, including Wrangler’s `--durable-objects-code-update-mode` flag.
- [Worker API reference](https://alchemy.run/providers/cloudflare/workers#worker) — every prop, including `codeUpdateStrategy`.
