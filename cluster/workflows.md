---
url: https://alchemy.run/cluster/workflows
title: "Workflows"
description: "Durable workflows on Effect Cluster — persisted activities, durable sleeps, signals, compensations and child workflows — declared and implemented like an entity and started from any runtime."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A workflow is a durable function. Each step, an **activity**, is persisted when it finishes, so a workflow that stops, because its runner restarts or a deploy replaces it, picks up where it left off on another runner without running finished steps again.

A run’s steps are written to the database as they finish, so the run doesn’t belong to any one runner. When it waits, for a signal or a sleep, it holds no runner at all, and whichever runner owns its shard when it wakes carries on:

<svg viewBox="0 0 655.2 168" style="max-width: 753px" role="img" aria-label="An Invite run waits for the guest's answer in the database. Runner 1 stops meanwhile, and the run finishes on Runner 2 without starting over."><line stroke-opacity="0.2" stroke="currentColor" x1="475.2" y1="24" x2="475.2" y2="162"></line><g><text fill="currentColor" x="4" y="56">Runner 1</text> <g><rect stroke="currentColor" fill="none" x="83.2" y="40" width="83.99999999999999" height="24" rx="5"></rect><text fill="currentColor" x="125.19999999999999" y="56" text-anchor="middle">start</text></g> <g><rect stroke="currentColor" fill="none" x="251.2" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="307.2" y="56" text-anchor="middle">stopped</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="655.2" y2="74"></line><text fill="currentColor" x="4" y="100">Database</text> <g><rect stroke="currentColor" fill="none" x="167.2" y="84" width="308" height="24" rx="5"></rect><text fill="currentColor" x="321.2" y="100" text-anchor="middle">waiting for Answer</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="655.2" y2="118"></line><text fill="currentColor" x="4" y="144">Runner 2</text> <g><rect stroke="currentColor" fill="none" x="475.2" y="128" width="112.00000000000006" height="24" rx="5"></rect><text fill="currentColor" x="531.2" y="144" text-anchor="middle">join</text></g></g> <text fill="currentColor" x="475.2" y="18" text-anchor="middle">answer: true</text></svg> <svg viewBox="0 0 330 198" style="max-width: 379px" role="img" aria-label="An Invite run waits for the guest's answer in the database. Runner 1 stops meanwhile, and the run finishes on Runner 2 without starting over."><line stroke-opacity="0.2" stroke="currentColor" x1="222.6" y1="24" x2="222.6" y2="192"></line><g><text fill="currentColor" x="0" y="44">Runner 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="47.699999999999996" height="24" rx="5"></rect><text fill="currentColor" x="23.849999999999998" y="70" text-anchor="middle">start</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="54" width="63.60000000000001" height="24" rx="5"></rect><text fill="currentColor" x="127.19999999999999" y="70" text-anchor="middle">stopped</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">Database</text> <g><rect stroke="currentColor" fill="none" x="47.699999999999996" y="108" width="174.9" height="24" rx="5"></rect><text fill="currentColor" x="135.15" y="124" text-anchor="middle">waiting for Answer</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">Runner 2</text> <g><rect stroke="currentColor" fill="none" x="222.6" y="162" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="254.39999999999998" y="178" text-anchor="middle">join</text></g></g> <text fill="currentColor" x="222.6" y="18" text-anchor="middle">answer: true</text></svg>

An Invite run waits for the guest's answer in the database. Runner 1 stops meanwhile, and the run finishes on Runner 2 without starting over.

## Declare and implement it

A workflow takes the same options as Effect’s `Workflow.make`: its payload, success and error, and the key that makes two starts of the same work one run. Its implementation comes after them, an Effect that resolves bindings once per runner and returns the handler:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Effect from "effect/Effect";
import * as Schema from "effect/Schema";
import { Activity, DurableDeferred } from "effect/workflow";
import { Default } from "./Cluster.ts";
import Room from "./Room.ts";

/** The guest's answer, which the run waits for. */
export const Answer = DurableDeferred.make("answer", { success: Schema.Boolean });

export class Declined extends Schema.TaggedError<Declined>()("Declined", {}) {}

export default class Invite extends Cluster.Workflow<Invite>()(
  "Invite",
  {
    payload: { room: Schema.String, guest: Schema.String },
    success: Schema.String,
    error: Declined,
    idempotencyKey: ({ room, guest }) => \`${room}/${guest}\`,
  },
  { main: import.meta.url },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return Effect.fn(function* ({ room, guest }) {
      const accepted = yield* DurableDeferred.await(Answer);
      if (!accepted) return yield* new Declined();
      const members = yield* Activity.make({
        name: "join",
        success: Schema.Int,
        execute: rooms.getByName(room).pipe(
          Effect.flatMap((target) => target.Join({ name: guest })),
          Effect.orDie,
        ),
      });
      return \`${guest} joined ${room} with ${members} others\`;
    });
  }).pipe(Effect.provide(Default)),
) {}
```

The handler’s payload, success and error are typed by the options. On the pool, each workflow runs on Effect Cluster’s own workflow engine, as a workflow entity per run.

## Keep its code out of its callers

As with an [entity](entities.md#keep-its-code-out-of-its-callers), a caller outside the workflow’s pool would bundle its implementation and its pool. For those callers, declare the workflow on its own:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Schema from "effect/Schema";
import { DurableDeferred } from "effect/workflow";

export const Answer = DurableDeferred.make("answer", { success: Schema.Boolean });

export class Declined extends Schema.TaggedError<Declined>()("Declined", {}) {}

export class Invite extends Cluster.Workflow<Invite>()("Invite", {
  payload: { room: Schema.String, guest: Schema.String },
  success: Schema.String,
  error: Declined,
  idempotencyKey: ({ room, guest }) => \`${room}/${guest}\`,
}) {}
```

Then implement it in a file of its own with `.make`, which returns a Layer, typing the handler with `Invite.of`:

```typescript
import * as Effect from "effect/Effect";
import * as Schema from "effect/Schema";
import { Activity, DurableDeferred } from "effect/workflow";
import { Default } from "./Cluster.ts";
import { Answer, Declined, Invite } from "./Invite.ts";
import { Room } from "./Room.ts";

export default Invite.make(
  { main: import.meta.url },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return Invite.of(
      Effect.fn(function* ({ room, guest }) {
        const accepted = yield* DurableDeferred.await(Answer);
        if (!accepted) return yield* new Declined();
        const members = yield* Activity.make({
          name: "join",
          success: Schema.Int,
          execute: rooms.getByName(room).pipe(
            Effect.flatMap((target) => target.Join({ name: guest })),
            Effect.orDie,
          ),
        });
        return \`${guest} joined ${room} with ${members} others\`;
      }),
    );
  }).pipe(Effect.provide(Default)),
);
```

## Start and read runs

`yield* Invite` from any runtime returns its client. `start` begins a run, or returns the run already started with the same key:

```typescript
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Answer, Invite } from "./Invite.ts";

export default class Api extends Docker.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const invites = yield* Invite;
    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const url = new URL(request.url, "http://api");
        if (url.pathname === "/invite") {
          const run = yield* invites.start({ room: "lobby", guest: "sam" });
          return yield* HttpServerResponse.json({ executionId: run.executionId });
        }
        const run = invites.get(url.searchParams.get("executionId") ?? "");
        if (url.pathname === "/accept") yield* run.signal(Answer, true);
        return yield* HttpServerResponse.json(yield* run.status);
      }).pipe(Effect.orDie),
    };
  }),
) {}
```

A run’s `status` is `NotStarted`, `Running`, `Completed` with its result, `Failed` with its error, `Interrupted` or `Died`. `signal` completes a `DurableDeferred` the run waits on, and `interrupt` stops it where it waits.

## Sleep durably

In a workflow’s handler, `DurableClock.sleep` from `effect/workflow` waits without holding a runner: the wake-up is a persisted message, delivered when it’s due. Here is the handler of a `Reminder` workflow, declared like `Invite` with a `room` in its payload:

```typescript
Effect.fn(function* ({ room }) {
  yield* DurableClock.sleep({ name: "wait", duration: "1 day" });
  yield* Activity.make({ name: "remind", execute: Effect.logInfo(\`remind ${room}\`) });
});
```

Runners pick up due messages every 10 seconds, so a sleep wakes up to 10 seconds late.

## Undo a step

A workflow class’s `withCompensation` attaches an undo to an activity. If the workflow fails later, the compensations run, newest first. Here is the handler of a `Checkout` workflow, with an `orderId` in its payload:

```typescript
Effect.fn(function* ({ orderId }) {
  yield* Activity.make({ name: "reserve", execute: Effect.logInfo(\`reserve ${orderId}\`) }).pipe(
    Checkout.withCompensation(() =>
      Activity.make({ name: "release", execute: Effect.logInfo(\`release ${orderId}\`) }),
    ),
  );
  yield* Activity.make({ name: "charge", execute: Effect.logInfo(\`charge ${orderId}\`) });
});
```

## Child workflows

Inside a workflow, the other workflow class’s `execute` runs it as a child and waits for it. Here an `Onboard` workflow, with a `guest` in its payload, runs `Invite` for the lobby:

```typescript
Effect.fn(function* ({ guest }) {
  return yield* Invite.execute({ room: "lobby", guest });
});
```

Run a child in the same pool as its parent.

## Run the same workflow on any engine

Code written against Alchemy’s engine-neutral `Workflows.RunWorkflow` runs a `Cluster.Workflow` with `Cluster.WorkflowEngine`:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Docker from "@alchemy.run/docker";
import * as Workflows from "alchemy/Workflows";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Invite } from "./Invite.ts";

export default class Api extends Docker.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const invite = yield* Workflows.RunWorkflow(Invite.workflow);
    return {
      fetch: Effect.gen(function* () {
        const run = yield* invite.start({ room: "lobby", guest: "sam" });
        return HttpServerResponse.text(run.executionId);
      }),
    };
  }).pipe(Effect.provide(Cluster.WorkflowEngine(Invite))),
) {}
```
