---
url: https://alchemy.run/cloudflare/messaging/cron
title: "Scheduled jobs with Cron Triggers"
description: "Run Worker code on a schedule — declare a cron expression with an Effect handler, deploy it, and prove the trigger fires with a bounded polling test."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Cron Triggers invoke a Worker on a schedule — nightly cleanups, periodic syncs, heartbeats. In alchemy, a single call declares both halves of a scheduled Worker: `Cloudflare.Workers.cron(expression, handler)` attaches the cron expression to the Worker at deploy time *and* registers the runtime `scheduled` listener that runs your Effect on each fire.

To run a [Workflow](../compute/workflows.md#schedule-a-workflow) on a schedule, attach `schedules` to the Workflow itself instead of wrapping `workflow.create()` in a `scheduled` handler.

## Declare a cron on the Worker

Call `cron` in the Worker’s Construction phase and provide `CronEventSourceLive` as a layer:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    yield* Cloudflare.Workers.cron("0 12 * * *", (controller) =>
      Effect.log(\`scheduled at ${controller.scheduledTime}\`),
    );

    return {
      fetch: Effect.gen(function* () {
        return HttpServerResponse.text("ok");
      }),
    };
  }).pipe(Effect.provide(Cloudflare.Workers.CronEventSourceLive)),
);
```

At deploy time the call binds `0 12 * * *` onto the Worker’s Cron Triggers; at runtime it registers a `scheduled` listener that runs your handler every day at noon UTC. The `controller` is Cloudflare’s `ScheduledController` — `controller.scheduledTime` is the fire time and `controller.cron` is the expression that fired.

## Multiple schedules

Register `cron` once per schedule; each handler only runs for fires of its own expression:

```typescript
yield* Cloudflare.Workers.cron("*/5 * * * *", () => syncFeeds);
yield* Cloudflare.Workers.cron("0 0 * * *", () => purgeExpired);
```

<svg viewBox="0 0 677.4 142" style="max-width: 779px" role="img" aria-label="Both expressions fire the same Worker's scheduled handler. At midnight both fire, and each fire runs only the handler registered for its expression."><line stroke-opacity="0.2" stroke="currentColor" x1="394.73333333333335" y1="24" x2="394.73333333333335" y2="118"></line><g><text fill="currentColor" x="4" y="50">*/5 * * * *</text> <text fill="currentColor" x="4" y="64">syncFeeds</text> <g><rect stroke="currentColor" fill="none" x="114.73333333333333" y="40" width="46.66666666666667" height="24" rx="5"></rect><text fill="currentColor" x="138.06666666666666" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="208.06666666666666" y="40" width="46.666666666666686" height="24" rx="5"></rect><text fill="currentColor" x="231.4" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="301.40000000000003" y="40" width="46.666666666666686" height="24" rx="5"></rect><text fill="currentColor" x="324.73333333333335" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="394.73333333333335" y="40" width="46.66666666666663" height="24" rx="5"></rect><text fill="currentColor" x="418.06666666666666" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="488.0666666666666" y="40" width="46.66666666666674" height="24" rx="5"></rect><text fill="currentColor" x="511.4" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="581.4" y="40" width="46.66666666666663" height="24" rx="5"></rect><text fill="currentColor" x="604.7333333333333" y="56" text-anchor="middle">run</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="677.4" y2="74"></line><text fill="currentColor" x="4" y="94">0 0 * * *</text> <text fill="currentColor" x="4" y="108">purgeExpired</text> <g><rect stroke="currentColor" fill="none" x="394.73333333333335" y="84" width="158.66666666666663" height="24" rx="5"></rect><text fill="currentColor" x="474.06666666666666" y="100" text-anchor="middle">purgeExpired</text></g></g> <text fill="currentColor" x="394.73333333333335" y="18" text-anchor="middle">00:00 UTC</text> <line stroke-opacity="0.2" stroke="currentColor" x1="105.4" y1="118" x2="665.4" y2="118"></line><text fill="currentColor" x="114.73333333333333" y="134" text-anchor="middle">23:50</text> <text fill="currentColor" x="394.73333333333335" y="134" text-anchor="middle">00:00</text> <text fill="currentColor" x="665.4" y="134" text-anchor="end">00:10</text></svg> <svg viewBox="0 0 330 162" style="max-width: 379px" role="img" aria-label="Both expressions fire the same Worker's scheduled handler. At midnight both fire, and each fire runs only the handler registered for its expression."><line stroke-opacity="0.2" stroke="currentColor" x1="164.3" y1="24" x2="164.3" y2="138"></line><g><text fill="currentColor" x="0" y="44">*/5 * * * * <tspan fill="currentColor" dx="8">syncFeeds</tspan></text> <g><rect stroke="currentColor" fill="none" x="5.3" y="54" width="26.499999999999996" height="24" rx="5"></rect><text fill="currentColor" x="37.8" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="58.300000000000004" y="54" width="26.499999999999993" height="24" rx="5"></rect><text fill="currentColor" x="90.8" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="111.30000000000001" y="54" width="26.5" height="24" rx="5"></rect><text fill="currentColor" x="143.8" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="164.3" y="54" width="26.49999999999997" height="24" rx="5"></rect><text fill="currentColor" x="196.79999999999998" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="217.29999999999998" y="54" width="26.5" height="24" rx="5"></rect><text fill="currentColor" x="249.79999999999998" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="270.3" y="54" width="26.499999999999943" height="24" rx="5"></rect><text fill="currentColor" x="302.79999999999995" y="70" text-anchor="start">run</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">0 0 * * * <tspan fill="currentColor" dx="8">purgeExpired</tspan></text> <g><rect stroke="currentColor" fill="none" x="164.3" y="108" width="90.09999999999997" height="24" rx="5"></rect><text fill="currentColor" x="209.35" y="124" text-anchor="middle">purgeExpired</text></g></g> <text fill="currentColor" x="164.3" y="18" text-anchor="middle">00:00 UTC</text> <line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="318" y2="138"></line><text fill="currentColor" x="5.3" y="154" text-anchor="middle">23:50</text> <text fill="currentColor" x="164.3" y="154" text-anchor="middle">00:00</text> <text fill="currentColor" x="318" y="154" text-anchor="end">00:10</text></svg>

Both expressions fire the same Worker's scheduled handler. At midnight both fire, and each fire runs only the handler registered for its expression.

The listener checks `controller.cron` against the expression it was registered with, so a fire of `0 0 * * *` never runs the `*/5` handler and vice versa.

## Deploy it

Wire the Worker into a Stack and expose its URL and attached crons as outputs:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import Worker from "./src/worker.ts";

export default Alchemy.Stack(
  "CronApp",
  {
    providers: Cloudflare.providers(),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const worker = yield* Worker;
    return {
      url: worker.url.as<string>(),
      crons: worker.crons,
    };
  }),
);
```

`worker.crons` is an output attribute listing every cron expression attached to the deployed Worker — the test below asserts against it.

```sh
pnpm alchemy deploy
```

## Test it: record each fire in a Durable Object

Cloudflare’s minimum cron granularity is one minute, so you can’t just sleep and assert — you need the handler to leave evidence you can poll for. A Durable Object that records each `scheduledTime` is perfect:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";

export default class CronCounter extends Cloudflare.DurableObject<CronCounter>()(
  "CronCounter",
  Effect.gen(function* () {
    const state = yield* Cloudflare.DurableObjectState;
    return Effect.gen(function* () {
      let times = (yield* state.storage.get<number[]>("times")) ?? [];
      return {
        record: Effect.fn(function* (time: number) {
          times = [...times, time];
          yield* state.storage.put("times", times);
        }),
        snapshot: () => Effect.succeed({ times }),
        reset: Effect.fn(function* () {
          times = [];
          yield* state.storage.put("times", times);
        }),
      };
    });
  }),
) {}
```

`record` appends a fire timestamp to transactional storage, `snapshot` returns everything recorded so far, and `reset` clears leftover state between test runs.

## Wire the cron to the counter

Point the cron at the DO and expose the recordings over HTTP. The trigger is `* * * * *` — every minute, the fastest schedule Cloudflare supports — so the test doesn’t wait long:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import CronCounter from "./cron-counter.ts";

export default Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    const counters = yield* CronCounter;

    yield* Cloudflare.Workers.cron("* * * * *", (controller) =>
      counters.getByName("default").record(controller.scheduledTime),
    );

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const url = new URL(request.url, "http://localhost");

        if (request.method === "GET" && url.pathname === "/times") {
          const snapshot = yield* counters.getByName("default").snapshot();
          return yield* HttpServerResponse.json(snapshot);
        }

        if (request.method === "POST" && url.pathname === "/reset") {
          yield* counters.getByName("default").reset();
          return yield* HttpServerResponse.json({ ok: true });
        }

        return HttpServerResponse.text("Not Found", { status: 404 });
      }),
    };
  }).pipe(Effect.provide(Cloudflare.Workers.CronEventSourceLive)),
);
```

Each fire records `controller.scheduledTime` on the `"default"` counter instance; `GET /times` reads it back and `POST /reset` wipes it so reruns start clean.

## Poll until it fires

The test deploys the Stack once, resets the counter, then polls `/times` with a bounded `Effect.repeat` until a fire newer than the reset shows up:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Test from "@alchemy.run/test/Vitest";
import { expect } from "@effect/vitest";
import * as Effect from "effect/Effect";
import * as Schedule from "effect/Schedule";
import * as HttpClient from "effect/http/HttpClient";
import Stack from "../alchemy.run.ts";

const { test, beforeAll, afterAll, deploy, destroy } = Test.make({
  providers: Cloudflare.providers(),
  state: Cloudflare.state(),
});

const stack = beforeAll(deploy(Stack));
afterAll.skipIf(!!process.env.NO_DESTROY)(destroy(Stack));

test(
  "the scheduled handler fires on its cron trigger",
  Effect.gen(function* () {
    const { url, crons } = yield* stack;
    expect(crons).toContain("* * * * *");

    const client = yield* HttpClient.HttpClient;

    // Reset leftover state from prior runs. Doubles as a readiness
    // probe — a fresh workers.dev URL takes a few seconds to start
    // serving 200s.
    yield* Effect.gen(function* () {
      const res = yield* client.post(\`${url}/reset\`);
      if (res.status !== 200) {
        return yield* Effect.fail(new Error(\`Worker not ready: ${res.status}\`));
      }
    }).pipe(
      Effect.retry({
        schedule: Schedule.exponential("500 millis"),
        times: 10,
      }),
    );
    const resetAt = Date.now();

    // Cron granularity is one minute and there's some propagation
    // delay after deploy, so poll up to ~3 minutes for the first fire.
    const times = yield* Effect.gen(function* () {
      const res = yield* client.get(\`${url}/times\`);
      if (res.status !== 200) return [];
      const body = (yield* res.json) as { times?: number[] };
      return (body.times ?? []).filter((t) => t >= resetAt);
    }).pipe(
      Effect.catch(() => Effect.succeed([])),
      Effect.repeat({
        schedule: Schedule.spaced("5 seconds"),
        until: (recent) => recent.length > 0,
        times: 36,
      }),
    );

    expect(times.length).toBeGreaterThan(0);
  }),
  { timeout: 240_000 },
);
```

The poll is declarative and bounded: `Schedule.spaced("5 seconds")` with `times: 36` caps it at ~3 minutes, `until` stops as soon as a recent fire appears, and `Effect.catch` treats transient fetch errors as “not yet” instead of failing the test. Never write a `while (Date.now() < deadline)` loop for this — it ignores interruption and hides intent.

## Where next

- [Workers](../compute/workers.md) — the two-phase Worker model that cron handlers live inside.
- [Durable Objects](../compute/durable-objects.md) — the stateful building block the test uses to record fires.
- [Workflows](../compute/workflows.md#schedule-a-workflow) — native cron schedules on the Workflow itself.
- [cron API reference](https://alchemy.run/providers/cloudflare/workers#cron)
