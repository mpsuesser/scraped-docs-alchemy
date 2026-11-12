---
url: https://alchemy.run/cluster/crons
title: "Crons"
description: "Run a job on a schedule with an Effect Cluster cron — once per scheduled time across the cluster, with the schedule on its Layer."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A cron runs a job on a schedule, once per scheduled time across the whole cluster, however many runners there are. Each run is a persisted message, so a run that’s due while its runner restarts still happens.

Monday's and Wednesday's runs happen once each, on whichever runner owns the cron. Tuesday's falls in an outage and is more than an hour old when the runners return, so it's skipped.

## Declare and implement it

A cron is declared by name, with its schedule and the job:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import { ClusterCron } from "effect/cluster";
import * as Effect from "effect/Effect";
import { Default } from "./Cluster.ts";

export default class Cleanup extends Cluster.Cron<Cleanup>()(
  "Cleanup",
  { main: import.meta.url, cron: "0 3 * * *", skipIfOlderThan: "1 hour" },
  Effect.gen(function* () {
    return Effect.gen(function* () {
      const scheduled = yield* ClusterCron.CurrentScheduledTime;
      yield* Effect.logInfo(\`cleanup for ${scheduled.epochMilliseconds}\`);
    });
  }).pipe(Effect.provide(Default)),
) {}
```

`ClusterCron.CurrentScheduledTime` is the time the run was scheduled for, not when it started.

Nothing calls a cron, so `yield* Cleanup` in the Stack to deploy it, as with a [singleton](singletons.md#deploy-it).

## The schedule

`cron` is a cron expression, read in UTC unless you set `timeZone`. A sixth, leading field schedules by the second: `"*/10 * * * * *"` runs every ten seconds.

`skipIfOlderThan` skips a run that’s due longer ago than that, as after an outage. `calculateNextRunFromPrevious` schedules each run from the previous run’s scheduled time rather than from when it finished.

An expression that doesn’t parse fails the deploy:

```text
Cluster.InvalidCron: Cluster.Cron 'Cleanup': Invalid number of segments in cron expression
```

## Declare it on its own

A cron can also be declared by name alone, with its schedule and job on `.make`, which returns a Layer:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";

export class Cleanup extends Cluster.Cron<Cleanup>()("Cleanup") {}
```

```typescript
import { ClusterCron } from "effect/cluster";
import * as Effect from "effect/Effect";
import { Cleanup } from "./Cleanup.ts";
import { Default } from "./Cluster.ts";

export default Cleanup.make(
  { main: import.meta.url, cron: "0 3 * * *", skipIfOlderThan: "1 hour" },
  Effect.gen(function* () {
    return Effect.gen(function* () {
      const scheduled = yield* ClusterCron.CurrentScheduledTime;
      yield* Effect.logInfo(\`cleanup for ${scheduled.epochMilliseconds}\`);
    });
  }).pipe(Effect.provide(Default)),
);
```

Provide the Layer to the Stack like any other implementation.
