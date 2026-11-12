---
url: https://alchemy.run/gcp/messaging/scheduler
title: "Cloud Scheduler"
description: "Run code on a cron schedule with GCP.CloudScheduler.consumeSchedule — the handler lives on your Cloud Run service and only Cloud Scheduler can call it."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

`consumeSchedule` runs a handler on your Cloud Run service on a cron schedule. Alchemy creates the Cloud Scheduler job and a private path on the service that only the scheduler can call.

## Add a schedule

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Monitor extends GCP.Function<Monitor>()(
  "Monitor",
  { main: import.meta.url },
  Effect.gen(function* () {
    yield* GCP.CloudScheduler.consumeSchedule(
      "Heartbeat",
      { schedule: "* * * * *", timeZone: "Etc/UTC" },
      (event) => Effect.log(\`tick ${event.scheduleTime}\`),
    );

    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }).pipe(Effect.provide(GCP.Run.ScheduleEventSource)),
) {}
```

`schedule` is standard cron. `event` carries the job name and the time the run was scheduled for.

## Several schedules on one service

Give each schedule its own name:

```typescript
yield* GCP.CloudScheduler.consumeSchedule(
  "Heartbeat",
  { schedule: "* * * * *", timeZone: "Etc/UTC" },
  recordHeartbeat,
);

yield* GCP.CloudScheduler.consumeSchedule(
  "DailyRollup",
  { schedule: "0 0 * * *", timeZone: "Etc/UTC" },
  rollUpYesterday,
);
```

## Public service, private schedules

`invokerIamDisabled: true` opens your `fetch` routes to everyone. The schedule path still rejects any request without a valid token from Cloud Scheduler.

<svg viewBox="0 0 677.4 124" style="max-width: 779px" role="img" aria-label="Two Cloud Scheduler jobs call the same service, each on a private path of its own. At midnight both fire, and the service runs both handlers side by side."><line stroke-opacity="0.2" stroke="currentColor" x1="368.6" y1="24" x2="368.6" y2="118"></line><g><text fill="currentColor" x="4" y="50">Heartbeat</text> <text fill="currentColor" x="4" y="64">* * * * *</text> <g><rect stroke="currentColor" fill="none" x="116.60000000000001" y="40" width="39.2" height="24" rx="5"></rect><text fill="currentColor" x="136.20000000000002" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="200.6" y="40" width="39.20000000000002" height="24" rx="5"></rect><text fill="currentColor" x="220.2" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="284.6" y="40" width="39.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="304.20000000000005" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="368.6" y="40" width="39.200000000000045" height="24" rx="5"></rect><text fill="currentColor" x="388.20000000000005" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="452.6" y="40" width="39.200000000000045" height="24" rx="5"></rect><text fill="currentColor" x="472.20000000000005" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="536.6" y="40" width="39.200000000000045" height="24" rx="5"></rect><text fill="currentColor" x="556.2" y="56" text-anchor="middle">run</text></g> <g><rect stroke="currentColor" fill="none" x="620.5999999999999" y="40" width="39.200000000000045" height="24" rx="5"></rect><text fill="currentColor" x="640.1999999999999" y="56" text-anchor="middle">run</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="677.4" y2="74"></line><text fill="currentColor" x="4" y="94">DailyRollup</text> <text fill="currentColor" x="4" y="108">0 0 * * *</text> <g><rect stroke="currentColor" fill="none" x="368.6" y="84" width="162.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="449.8" y="100" text-anchor="middle">rollUpYesterday</text></g></g> <text fill="currentColor" x="368.6" y="18" text-anchor="middle">00:00 UTC</text></svg> <svg viewBox="0 0 330 174" style="max-width: 379px" role="img" aria-label="Two Cloud Scheduler jobs call the same service, each on a private path of its own. At midnight both fire, and the service runs both handlers side by side."><line stroke-opacity="0.2" stroke="currentColor" x1="149.46" y1="24" x2="149.46" y2="168"></line><g><text fill="currentColor" x="0" y="44">Heartbeat <tspan fill="currentColor" dx="8">* * * * *</tspan></text> <g><rect stroke="currentColor" fill="none" x="6.36" y="54" width="22.259999999999998" height="24" rx="5"></rect><text fill="currentColor" x="34.62" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="54.059999999999995" y="54" width="22.259999999999998" height="24" rx="5"></rect><text fill="currentColor" x="82.32" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="101.76" y="54" width="22.260000000000005" height="24" rx="5"></rect><text fill="currentColor" x="130.02" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="149.46" y="54" width="22.25999999999999" height="24" rx="5"></rect><text fill="currentColor" x="177.72" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="197.16" y="54" width="22.26000000000002" height="24" rx="5"></rect><text fill="currentColor" x="225.42000000000002" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="244.86" y="54" width="22.25999999999999" height="24" rx="5"></rect><text fill="currentColor" x="273.12" y="70" text-anchor="start">run</text></g> <g><rect stroke="currentColor" fill="none" x="292.56" y="54" width="22.25999999999999" height="24" rx="5"></rect><text fill="currentColor" x="292.56" y="90" text-anchor="start">run</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="99" x2="330" y2="99"></line><text fill="currentColor" x="0" y="113">DailyRollup <tspan fill="currentColor" dx="8">0 0 * * *</tspan></text> <g><rect stroke="currentColor" fill="none" x="149.46" y="123" width="92.22" height="24" rx="5"></rect><text fill="currentColor" x="149.46" y="159" text-anchor="start">rollUpYesterday</text></g></g> <text fill="currentColor" x="149.46" y="18" text-anchor="middle">00:00 UTC</text></svg>

Two Cloud Scheduler jobs call the same service, each on a private path of its own. At midnight both fire, and the service runs both handlers side by side.

## Use bindings in the handler

Handlers use the same clients as `fetch`:

```typescript
const writer = yield* GCP.BigQuery.WriteTable(table);

yield* GCP.CloudScheduler.consumeSchedule(
  "Heartbeat",
  { schedule: "* * * * *", timeZone: "Etc/UTC" },
  (event) =>
    writer
      .insert([{ jobName: event.jobName, receivedAt: new Date() }])
      .pipe(Effect.orDie),
);
```

## Start a job on a schedule

To run a [Cloud Run job](../compute/jobs.md) nightly, point a `CloudScheduler.Job` straight at it. See [Run it on a schedule](../compute/jobs.md#run-it-on-a-schedule).

## Example

[`examples/gcp-cron`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-cron) runs two schedules on one service and records each run in BigQuery.

## Where next

- [Cloud Run jobs](../compute/jobs.md) — batch work on a timer.
- [CloudScheduler reference](https://alchemy.run/providers/gcp/cloudscheduler#job) — every prop.
