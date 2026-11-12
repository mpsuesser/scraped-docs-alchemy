---
url: https://alchemy.run/gcp/compute/jobs
title: "Cloud Run jobs"
description: "Run batch work to completion with GCP.Run.Job — trigger it from a service with RunJob, or nightly with Cloud Scheduler."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

`GCP.Run.Job` runs an Effect program that starts, does its work, and exits. Nothing is billed between runs. Use it for nightly reports, backfills, and draining a queue in batches.

## Declare a job

A job returns a `run` Effect instead of `fetch`:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import { Reports } from "./resources.ts";

export default class Summarize extends GCP.Run.Job<Summarize>()(
  "Summarize",
  { main: import.meta.url },
  Effect.gen(function* () {
    const reports = yield* GCP.Storage.WriteBucket(Reports);

    return {
      run: Effect.gen(function* () {
        const day = new Date().toISOString().slice(0, 10);
        yield* reports.put(\`daily/${day}.json\`, JSON.stringify({ day }), {
          contentType: "application/json",
        });
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(GCP.Storage.WriteBucketHttp)),
) {}
```

Bindings work the same as on a service. The job gets its own service account and only the roles its bindings ask for.

## Executions and tasks

Each time a job starts, it creates an **execution**. An execution runs `taskCount` copies of your program, called **tasks**, `parallelism` at a time. Each task reads its index from `CLOUD_RUN_TASK_INDEX`, so it can take its own share of the work. A task that fails is retried on its own, and the execution succeeds when every task has.

<svg viewBox="0 0 640.4 212" style="max-width: 736px" role="img" aria-label="An execution with taskCount 4 and parallelism 2. Two tasks run at a time. Task 1's failed attempt is retried in the next free slot, and the execution succeeds when all four have."><line stroke-opacity="0.2" stroke="currentColor" x1="68.4" y1="24" x2="68.4" y2="206"></line><line stroke-opacity="0.2" stroke="currentColor" x1="572.4" y1="24" x2="572.4" y2="206"></line><g><text fill="currentColor" x="4" y="56">task 0</text> <g><rect stroke="currentColor" fill="none" x="68.4" y="40" width="190.39999999999995" height="24" rx="5"></rect><text fill="currentColor" x="163.59999999999997" y="56" text-anchor="middle">attempt 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="640.4" y2="74"></line><text fill="currentColor" x="4" y="100">task 1</text> <g><rect stroke="currentColor" fill="none" x="68.4" y="84" width="112" height="24" rx="5"></rect><text fill="currentColor" x="124.4" y="100" text-anchor="middle">fails</text></g> <g><rect stroke="currentColor" fill="none" x="270" y="84" width="168" height="24" rx="5"></rect><text fill="currentColor" x="354" y="100" text-anchor="middle">attempt 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="640.4" y2="118"></line><text fill="currentColor" x="4" y="144">task 2</text> <g><rect stroke="currentColor" fill="none" x="186" y="128" width="184.80000000000007" height="24" rx="5"></rect><text fill="currentColor" x="278.40000000000003" y="144" text-anchor="middle">attempt 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="162" x2="640.4" y2="162"></line><text fill="currentColor" x="4" y="188">task 3</text> <g><rect stroke="currentColor" fill="none" x="382" y="172" width="190.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="477.2" y="188" text-anchor="middle">attempt 1</text></g></g> <text fill="currentColor" x="68.4" y="18" text-anchor="middle">execution starts</text> <text fill="currentColor" x="572.4" y="18" text-anchor="middle">succeeds</text></svg> <svg viewBox="0 0 330 252" style="max-width: 379px" role="img" aria-label="An execution with taskCount 4 and parallelism 2. Two tasks run at a time. Task 1's failed attempt is retried in the next free slot, and the execution succeeds when all four have."><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="24" x2="0" y2="246"></line><line stroke-opacity="0.2" stroke="currentColor" x1="286.2" y1="24" x2="286.2" y2="246"></line><g><text fill="currentColor" x="0" y="44">task 0</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="108.11999999999999" height="24" rx="5"></rect><text fill="currentColor" x="54.059999999999995" y="70" text-anchor="middle">attempt 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">task 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="108" width="63.6" height="24" rx="5"></rect><text fill="currentColor" x="31.8" y="124" text-anchor="middle">fails</text></g> <g><rect stroke="currentColor" fill="none" x="114.47999999999999" y="108" width="95.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="162.17999999999998" y="124" text-anchor="middle">attempt 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">task 2</text> <g><rect stroke="currentColor" fill="none" x="66.78" y="162" width="104.94" height="24" rx="5"></rect><text fill="currentColor" x="119.25" y="178" text-anchor="middle">attempt 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="192" x2="330" y2="192"></line><text fill="currentColor" x="0" y="206">task 3</text> <g><rect stroke="currentColor" fill="none" x="178.07999999999998" y="216" width="108.12" height="24" rx="5"></rect><text fill="currentColor" x="232.14" y="232" text-anchor="middle">attempt 1</text></g></g> <text fill="currentColor" x="0" y="18" text-anchor="start">execution starts</text> <text fill="currentColor" x="286.2" y="18" text-anchor="middle">succeeds</text></svg>

An execution with taskCount 4 and parallelism 2. Two tasks run at a time. Task 1's failed attempt is retried in the next free slot, and the execution succeeds when all four have.

```typescript
export default class Reindex extends GCP.Run.Job<Reindex>()(
  "Reindex",
  { main: import.meta.url },
  { main: import.meta.url, taskCount: 4, parallelism: 2 },
```

## Retries and timeouts

```typescript
export default class Summarize extends GCP.Run.Job<Summarize>()(
  "Summarize",
  { main: import.meta.url },
  { main: import.meta.url, maxRetries: 1, timeout: "300s" },
```

A failed run retries `maxRetries` times, then the execution fails. `timeout` caps each attempt.

## Start it from a service

`RunJob` gives a service permission to start the job and returns a function that does it:

```typescript
import Summarize from "./Summarize.ts";

export default class Admin extends GCP.Function<Admin>()(
  "Admin",
  { main: import.meta.url },
  Effect.gen(function* () {
    const runSummarize = yield* GCP.Run.RunJob(Summarize);

    return {
      fetch: Effect.gen(function* () {
        const operation = yield* runSummarize();
        return yield* HttpServerResponse.json(
          { operation: operation.name },
          { status: 202 },
        );
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(GCP.Run.RunJobHttp)),
) {}
```

`runSummarize()` returns as soon as the execution starts, not when it finishes.

## Run it on a schedule

A Cloud Scheduler job can start the job directly, with no service in between. Give the job’s own account permission to run it, then point the scheduler at the Cloud Run API:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Output from "alchemy/Output";
import * as Effect from "effect/Effect";
import Summarize from "./Summarize.ts";

export const Nightly = Effect.gen(function* () {
  const job = yield* Summarize;

  const runner = yield* GCP.IAM.Member("NightlyRunner", {
    kind: "run.job",
    name: job.name,
    role: "roles/run.invoker",
    member: Output.interpolate\`serviceAccount:${job.serviceAccount}\`,
  });

  return yield* GCP.CloudScheduler.Job("Nightly", {
    schedule: "0 2 * * *",
    timeZone: "Etc/UTC",
    httpTarget: {
      uri: Output.interpolate\`https://run.googleapis.com/v2/${job.name}:run\`,
      httpMethod: "POST",
      body: "{}",
      oauthToken: {
        serviceAccountEmail: Output.map(runner.member, (member) =>
          member.replace(/^serviceAccount:/, ""),
        ),
        scope: "https://www.googleapis.com/auth/cloud-platform",
      },
    },
  });
});
```

Reading `runner.member` makes the scheduler wait for the grant. Yield `Nightly` from your Stack.

## Read the execution name

Cloud Run sets `CLOUD_RUN_EXECUTION` in the job’s environment. Log it to find a run in the console:

```typescript
const execution = yield* Effect.sync(() => process.env.CLOUD_RUN_EXECUTION);
```

## Example

[`examples/gcp-scheduled-job`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-scheduled-job) rolls BigQuery orders into a daily JSON summary in Cloud Storage, nightly and on demand.

## Where next

- [Pub/Sub](../messaging/pubsub.md) — a job that drains a subscription into BigQuery.
- [Cloud Scheduler](../messaging/scheduler.md) — schedules that call a service.
- [Run reference](https://alchemy.run/providers/gcp/run#job) — every prop and attribute.
