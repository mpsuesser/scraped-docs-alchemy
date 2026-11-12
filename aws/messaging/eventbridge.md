---
url: https://alchemy.run/aws/messaging/eventbridge
title: "EventBridge & Scheduler"
description: "How EventBridge buses route events to targets by matching their content, how runtimes publish and consume events, and how EventBridge Scheduler runs Functions, Queues and tasks on a schedule."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

EventBridge routes **events**. A producer puts an event on a **bus**, and **rules** on the bus look at its content and send it to whichever **targets** match. Producers don’t know the targets, and targets don’t know each other, so new reactions to an event are added as new rules.

AWS services publish their own events to every account’s default bus: an object created, an instance stopped, a task that failed. **EventBridge Scheduler** is the same idea driven by time: it invokes a target on a rate, a cron expression or a single date.

## Rules match events by content

Every event has a `source`, a `detail-type` and a JSON `detail`. A rule’s **event pattern** lists the values it matches, and a matching event goes to the rule’s targets:

<svg viewBox="0 0 763 242" style="max-width: 877px" role="img" aria-label="Each rule on the bus tests the event against its pattern. This OrderPlaced event matches two rules, so both of their targets get it, and the third rule ignores it."><defs><marker id="alc-fan-arrow-t246wc" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="270" y="8" width="282" height="226" rx="10"></rect><text fill="currentColor" x="280" y="26">Orders · event bus</text></g> <rect stroke="currentColor" fill="none" x="8" y="81.5" width="210" height="79" rx="8"></rect><text fill="currentColor" x="20" y="101.5">event</text> <text fill="currentColor" x="20" y="121.5" xml:space="preserve">"source": "shop.orders"</text> <text fill="currentColor" x="20" y="136.5" xml:space="preserve">"detail-type": "OrderPlaced"</text> <text fill="currentColor" x="20" y="151.5" xml:space="preserve">"detail": { "total": 120 }</text> <g><path stroke="currentColor" fill="none" d="M 218 121 C 249 121, 249 64, 280 64" marker-end="url(#alc-fan-arrow-t246wc)"></path><g><rect stroke="currentColor" fill="none" x="282" y="38" width="258" height="52" rx="7"></rect><text fill="currentColor" x="294" y="55">rule</text> <text fill="currentColor" x="294" y="72.5" xml:space="preserve">{ "detail-type": ["OrderPlaced"] }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="540" y1="64" x2="626" y2="64" marker-end="url(#alc-fan-arrow-t246wc)"></line></g><rect stroke="currentColor" fill="none" x="628" y="41" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="691.5" y="61" text-anchor="middle" opacity="1">Worker</text> <text fill="currentColor" x="691.5" y="77" text-anchor="middle">Lambda Function</text></g> <g><path stroke="currentColor" fill="none" d="M 218 121 C 249 121, 249 130, 280 130" marker-end="url(#alc-fan-arrow-t246wc)"></path><g><rect stroke="currentColor" fill="none" x="282" y="104" width="258" height="52" rx="7"></rect><text fill="currentColor" x="294" y="121">rule</text> <text fill="currentColor" x="294" y="138.5" xml:space="preserve">{ "source": ["shop.orders"] }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="540" y1="130" x2="626" y2="130" marker-end="url(#alc-fan-arrow-t246wc)"></line></g><rect stroke="currentColor" fill="none" x="628" y="107" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="691.5" y="127" text-anchor="middle" opacity="1">Audit</text> <text fill="currentColor" x="691.5" y="143" text-anchor="middle">SQS Queue</text></g> <g><path stroke="currentColor" fill="none" d="M 218 121 C 249 121, 249 196, 280 196"></path><g><rect stroke="currentColor" fill="none" x="282" y="170" width="258" height="52" rx="7"></rect><text fill="currentColor" x="294" y="187">rule</text> <text fill="currentColor" x="294" y="204.5" xml:space="preserve">{ "detail-type": ["OrderRefunded"] }</text> <text fill="currentColor" x="552" y="200">no match</text></g> <rect stroke="currentColor" fill="none" x="628" y="173" width="127" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="691.5" y="193" text-anchor="middle" opacity="0.6">Refunds</text> <text fill="currentColor" x="691.5" y="209" text-anchor="middle">Lambda Function</text></g></svg> <svg viewBox="0 0 318 561" style="max-width: 366px" role="img" aria-label="Each rule on the bus tests the event against its pattern. This OrderPlaced event matches two rules, so both of their targets get it, and the third rule ignores it."><defs><marker id="alc-fan-arrow-n-t246wc" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="14" y="93" width="304" height="466" rx="10"></rect><text fill="currentColor" x="24" y="111">Orders · event bus</text></g> <rect stroke="currentColor" fill="none" x="0" y="0" width="210" height="79" rx="8"></rect><text fill="currentColor" x="12" y="20">event</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">"source": "shop.orders"</text> <text fill="currentColor" x="12" y="55" xml:space="preserve">"detail-type": "OrderPlaced"</text> <text fill="currentColor" x="12" y="70" xml:space="preserve">"detail": { "total": 120 }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="79" x2="8" y2="449"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="157" x2="20" y2="157" marker-end="url(#alc-fan-arrow-n-t246wc)"></line><g><rect stroke="currentColor" fill="none" x="22" y="131" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="148">rule</text> <text fill="currentColor" x="34" y="165.5" xml:space="preserve">{ "detail-type": ["OrderPlaced"] }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="183" x2="166" y2="211" marker-end="url(#alc-fan-arrow-n-t246wc)"></line></g><rect stroke="currentColor" fill="none" x="102.5" y="213" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="233" text-anchor="middle" opacity="1">Worker</text> <text fill="currentColor" x="166" y="249" text-anchor="middle">Lambda Function</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="303" x2="20" y2="303" marker-end="url(#alc-fan-arrow-n-t246wc)"></line><g><rect stroke="currentColor" fill="none" x="22" y="277" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="294">rule</text> <text fill="currentColor" x="34" y="311.5" xml:space="preserve">{ "source": ["shop.orders"] }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="329" x2="166" y2="357" marker-end="url(#alc-fan-arrow-n-t246wc)"></line></g><rect stroke="currentColor" fill="none" x="102.5" y="359" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="379" text-anchor="middle" opacity="1">Audit</text> <text fill="currentColor" x="166" y="395" text-anchor="middle">SQS Queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="449" x2="20" y2="449"></line><g><rect stroke="currentColor" fill="none" x="22" y="423" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="440">rule</text> <text fill="currentColor" x="34" y="457.5" xml:space="preserve">{ "detail-type": ["OrderRefunded"] }</text> <text fill="currentColor" x="166" y="494" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="102.5" y="505" width="127" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="525" text-anchor="middle" opacity="0.6">Refunds</text> <text fill="currentColor" x="166" y="541" text-anchor="middle">Lambda Function</text></g></svg>

Each rule on the bus tests the event against its pattern. This OrderPlaced event matches two rules, so both of their targets get it, and the third rule ignores it.

A pattern can match on any field of the event, including fields inside `detail`, so a rule can take only the orders over a certain amount, or only the ones from one region. Events that match no rule are dropped.

## Declare a bus

```typescript
import * as AWS from "@alchemy.run/aws";

export const Orders = AWS.EventBridge.EventBus("Orders", {});
```

Your own events go on a bus of your own. The account’s **default bus**, where AWS services publish, needs no declaration: leave the bus out of the calls below to use it.

## Publish events

Bind `EventBridge.PutEvents` to the bus. Its role gets `events:PutEvents` on that bus only:

```typescript
const bus = yield* Orders;
const putEvents = yield* AWS.EventBridge.PutEvents(bus);

yield* putEvents({
  Entries: [{ Source: "app.orders", DetailType: "OrderPlaced", Detail: JSON.stringify({ orderId }) }],
});
```

Provide `AWS.EventBridge.PutEventsHttp` to the runtime. The bus name is filled in from the binding.

## Consume events in a Function

Subscribe a Lambda Function to a pattern from Construction. Matching events arrive as a `Stream`:

```typescript
export default class Worker extends AWS.Lambda.Function<Worker>()(
  "Worker",
  { main: import.meta.url, functionUrl: false },
  Effect.gen(function* () {
    const bus = yield* Orders;
    yield* AWS.EventBridge.consumeBusEvents(
      bus,
      { source: ["app.orders"], "detail-type": ["OrderPlaced"] },
      (events) => Stream.runForEach(events, (event) => Effect.log(event.detail)),
    );
    return {};
  }).pipe(Effect.provide(AWS.Lambda.EventSource)),
) {}
```

Deploy creates the rule and the Lambda permission that lets EventBridge invoke the Function. Each event’s `detail` is `unknown` until you narrow the stream to the type you expect, `AWS.EventBridge.EventRecord<OrderPlaced>`.

## Route events without code

Some targets need no handler of your own: a Queue to buffer events, an ECS task to process them. `EventBridge.events` builds the rule and sends matches straight to the target:

```typescript
yield* AWS.EventBridge.events(bus, { source: ["app.orders"] }).toQueue(audit);
```

`.toQueue` adds a statement to the Queue’s policy letting this rule send to it. `.toLambda` adds the invoke permission, and `.toEcsTask` creates a role that may run the task. Each grant is limited to the one rule.

For input transformers, retry policies, dead-letter queues or several targets on one rule, declare an `AWS.EventBridge.Rule` yourself. A raw rule grants nothing, so give its targets a role or a resource policy.

## React to AWS’s own events

AWS services publish to the default bus whenever something happens in your account. Match their `source` to react to them, leaving the bus out:

```typescript
yield* AWS.EventBridge.consumeBusEvents(
  { source: ["aws.ecs"], "detail-type": ["ECS Task State Change"] },
  (events) => Stream.runForEach(events, (event) => Effect.log(event.detail)),
);
```

This is how you watch for failed tasks, stopped instances or new objects without polling anything.

## Run things on a schedule

EventBridge Scheduler invokes a target on a schedule. `Scheduler.every` takes a plain duration:

```typescript
const report = yield* Report;
yield* AWS.Scheduler.every("5 minutes").toLambda(report);
```

The schedule needs a role that Scheduler can assume and that may invoke the Function. The builder creates it, with `lambda:InvokeFunction` on that Function only.

For a time of day, use `cron` with a timezone, and pass an input:

```typescript
yield* AWS.Scheduler.cron("cron(0 3 * * ? *)", { timezone: "America/New_York" })
  .named("nightly-report")
  .toLambda(report, { input: { job: "nightly-report" } });
```

For a single moment, use `at`. Schedules can target Queues too:

```typescript
yield* AWS.Scheduler.at(new Date("2026-08-01T09:00:00Z")).toQueue(reminders, { type: "launch" });
```

## Schedule containers

A job that runs too long for Lambda runs as an ECS task on a schedule instead:

```typescript
yield* AWS.ECS.every("NightlyJob", "cron(0 3 * * ? *)", {
  cluster,
  task: nightlyTask,
  subnets: network.privateSubnetIds,
  securityGroups: [jobSecurityGroup.groupId],
});
```

The schedule gets a role that may run the task and pass its roles. See [ECS](../compute/ecs.md).

## EventBridge, SNS or SQS

Use EventBridge when many kinds of events share one bus and different parts of the app care about different ones, or when you react to AWS’s own events and schedules. Use an [SNS Topic](sns.md) for one channel whose every message goes to every subscriber. Use an [SQS Queue](sqs.md) to buffer work for one consumer.
