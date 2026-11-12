---
url: https://alchemy.run/cloudflare/messaging/queues
title: "Queues"
description: "How Cloudflare Queues carry messages from producer Workers to a consumer in batches, with retries and dead-letter queues, and how platform events like Workflow completions arrive on a Queue."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Queue carries messages from one Worker to another. The **producer** sends a message and returns at once. Cloudflare holds it, and delivers it in a **batch** to the **consumer** Worker, which does the work. A slow job no longer slows the request that asked for it, and a burst of work waits in the Queue instead of overwhelming the consumer.

## Delivered at least once, in batches

The consumer receives messages in batches, a few or a hundred at a time. When it finishes a batch, the messages are **acknowledged** and removed. When it fails, they’re **retried** later. After too many attempts, a message moves to a **dead-letter queue**, if the Queue has one, or is dropped.

<svg viewBox="-4 28 716.6 200" style="max-width: 824px" role="img" aria-label="A message leaves only when its batch succeeds. A failed batch comes back after a delay, and a message that keeps failing moves to the dead-letter queue."><defs><marker id="alc-sm-arrow-we88bl" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 180.2 63 Q 240.2 63 296.19999999999993 63" marker-end="url(#alc-sm-arrow-we88bl)"></path><path stroke="currentColor" fill="none" d="M 440.4 63 Q 500.4 63 556.4 63" marker-end="url(#alc-sm-arrow-we88bl)"></path><path stroke="currentColor" fill="none" d="M 338.09999999999997 86 Q 279.29999999999995 128 334.84506611517304 167.67504722512362" marker-end="url(#alc-sm-arrow-we88bl)"></path><path stroke="currentColor" fill="none" d="M 402.49999999999994 170 Q 461.29999999999995 128 405.75493388482687 88.32495277487638" marker-end="url(#alc-sm-arrow-we88bl)"></path><path stroke="currentColor" fill="none" d="M 440.4 193 Q 500.4 193 556.4 193" marker-end="url(#alc-sm-arrow-we88bl)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-we88bl)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="67" text-anchor="middle">queued</text></g> <g><rect stroke="currentColor" fill="none" x="300.19999999999993" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="370.29999999999995" y="61" text-anchor="middle">in a batch</text> <text fill="currentColor" x="370.29999999999995" y="76" text-anchor="middle">with the consumer</text></g> <g><rect stroke="currentColor" fill="none" x="560.4" y="40" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="564.4" y="44" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="630.5" y="61" text-anchor="middle">acknowledged</text> <text fill="currentColor" x="630.5" y="76" text-anchor="middle">removed</text></g> <g><rect stroke="currentColor" fill="none" x="300.19999999999993" y="170" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="370.29999999999995" y="191" text-anchor="middle">waiting</text> <text fill="currentColor" x="370.29999999999995" y="206" text-anchor="middle">retryDelay</text></g> <g><rect stroke="currentColor" fill="none" x="560.4" y="170" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="564.4" y="174" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="630.5" y="191" text-anchor="middle">DeadLetters</text> <text fill="currentColor" x="630.5" y="206" text-anchor="middle">dead-letter queue</text></g> <g><rect stroke="currentColor" fill="none" x="204.49999999999997" y="53" width="69.4" height="17" rx="4"></rect><text fill="currentColor" x="239.19999999999996" y="66" text-anchor="middle">delivered</text></g> <g><rect stroke="currentColor" fill="none" x="448.2" y="53" width="102.39999999999999" height="17" rx="4"></rect><text fill="currentColor" x="499.4" y="66" text-anchor="middle">batch succeeds</text></g> <g><rect stroke="currentColor" fill="none" x="266.5862665287932" y="117.41876180628091" width="82.6" height="17" rx="4"></rect><text fill="currentColor" x="307.8862665287932" y="130.4187618062809" text-anchor="middle">batch fails</text></g> <g><rect stroke="currentColor" fill="none" x="404.6137334712067" y="118.58123819371909" width="56.199999999999996" height="17" rx="4"></rect><text fill="currentColor" x="432.7137334712067" y="131.5812381937191" text-anchor="middle">retried</text></g> <g><rect stroke="currentColor" fill="none" x="441.59999999999997" y="183" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="499.4" y="196" text-anchor="middle">after maxRetries</text></g></svg> <svg viewBox="28 -4 397 378" style="max-width: 457px" role="img" aria-label="A message leaves only when its batch succeeds. A failed batch comes back after a delay, and a message that keeps failing moves to the dead-letter queue."><defs><marker id="alc-sm-arrow-n-we88bl" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 110.1 86 Q 110.1 132 110.1 174" marker-end="url(#alc-sm-arrow-n-we88bl)"></path><path stroke="currentColor" fill="none" d="M 110.1 224 Q 110.1 270 110.1 312" marker-end="url(#alc-sm-arrow-n-we88bl)"></path><path stroke="currentColor" fill="none" d="M 120.36785714285713 178 Q 226.5 -59.73599999999993 331.00153949564725 174.34744847024996" marker-end="url(#alc-sm-arrow-n-we88bl)"></path><path stroke="currentColor" fill="none" d="M 332.6321428571428 224 Q 226.5 461.73599999999993 121.99846050435269 227.65255152975004" marker-end="url(#alc-sm-arrow-n-we88bl)"></path><path stroke="currentColor" fill="none" d="M 342.9 224 Q 342.9 270 342.9 312" marker-end="url(#alc-sm-arrow-n-we88bl)"></path><g><g><circle stroke="currentColor" fill="none" cx="110.1" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="110.1" y1="20" x2="110.1" y2="36" marker-end="url(#alc-sm-arrow-n-we88bl)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="67" text-anchor="middle">queued</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="199" text-anchor="middle">in a batch</text> <text fill="currentColor" x="110.1" y="214" text-anchor="middle">with the consumer</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="316" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="44" y="320" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="110.1" y="337" text-anchor="middle">acknowledged</text> <text fill="currentColor" x="110.1" y="352" text-anchor="middle">removed</text></g> <g><rect stroke="currentColor" fill="none" x="272.79999999999995" y="178" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="342.9" y="199" text-anchor="middle">waiting</text> <text fill="currentColor" x="342.9" y="214" text-anchor="middle">retryDelay</text></g> <g><rect stroke="currentColor" fill="none" x="272.79999999999995" y="316" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="276.79999999999995" y="320" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="342.9" y="337" text-anchor="middle">DeadLetters</text> <text fill="currentColor" x="342.9" y="352" text-anchor="middle">dead-letter queue</text></g> <g><rect stroke="currentColor" fill="none" x="75.39999999999999" y="121" width="69.4" height="17" rx="4"></rect><text fill="currentColor" x="110.1" y="134" text-anchor="middle">delivered</text></g> <g><rect stroke="currentColor" fill="none" x="58.9" y="259" width="102.39999999999999" height="17" rx="4"></rect><text fill="currentColor" x="110.1" y="272" text-anchor="middle">batch succeeds</text></g> <g><rect stroke="currentColor" fill="none" x="143.49234915962612" y="48.21886211756252" width="82.6" height="17" rx="4"></rect><text fill="currentColor" x="221.0923491596261" y="61.21886211756252" text-anchor="end">batch fails</text></g> <g><rect stroke="currentColor" fill="none" x="226.90765084037386" y="333.78113788243746" width="56.199999999999996" height="17" rx="4"></rect><text fill="currentColor" x="231.90765084037386" y="346.78113788243746" text-anchor="start">retried</text></g> <g><rect stroke="currentColor" fill="none" x="285.09999999999997" y="259" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="342.9" y="272" text-anchor="middle">after maxRetries</text></g></svg>

A message leaves only when its batch succeeds. A failed batch comes back after a delay, and a message that keeps failing moves to the dead-letter queue.

So a message is never lost to a failed consumer, but it can be delivered more than once. Make the work safe to repeat: check whether it’s done already, or write results under a key derived from the message.

## Declare a Queue

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Jobs = Cloudflare.Queues.Queue("Jobs");
```

## Send messages

Bind `WriteQueue` in the producer to get a sender. A message body is any value that can be serialized:

```typescript
export default Cloudflare.Worker(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const jobs = yield* Cloudflare.Queues.WriteQueue(Jobs);
    return {
      fetch: jobs.send({ type: "thumbnail", key: "avatars/sam.png" }).pipe(
        Effect.as(HttpServerResponse.empty({ status: 202 })),
        Effect.orDie,
      ),
    };
  }).pipe(Effect.provide(Cloudflare.Queues.WriteQueueBinding)),
);
```

`sendBatch` sends up to a hundred messages in one call.

## Consume messages

The consumer subscribes from its Construction phase. Each batch arrives as a `Stream` of messages:

```typescript
export default Cloudflare.Worker(
  "Consumer",
  { main: import.meta.url },
  Effect.gen(function* () {
    const jobs = yield* Jobs;
    yield* Cloudflare.Queues.consumeQueueMessages<Job>(jobs, (messages) =>
      Stream.runForEach(messages, (message) => Effect.log(message.body.key)),
    );
    return {};
  }).pipe(Effect.provide(Cloudflare.Queues.EventSourceLive)),
);
```

Deploy registers the Worker as the Queue’s consumer. When the handler succeeds, every message in the batch is acknowledged. When it fails, every message is retried. To decide per message, call `message.ack()` or `message.retry()` inside the handler.

One Worker can produce to a Queue and consume from it, or consume several Queues.

## Tune batches and retries

Pass settings before the handler:

```typescript
yield* Cloudflare.Queues.consumeQueueMessages<Job>(
  jobs,
  { batchSize: 25, maxWaitTime: "5 seconds", maxRetries: 3, retryDelay: "30 seconds" },
  handle,
);
```

A batch is delivered when it reaches `batchSize` or has waited `maxWaitTime`, whichever comes first. Larger batches mean fewer invocations. A shorter wait means lower latency. `retryDelay` spaces out the retries of a failed batch.

## Catch messages that keep failing

A message that fails `maxRetries` times is dropped, unless the Queue names a **dead-letter queue**. Cloudflare doesn’t alert on a dead-letter queue, so consume it too:

```typescript
const deadLetters = yield* DeadLetters;

yield* Cloudflare.Queues.consumeQueueMessages<Job>(
  jobs,
  { maxRetries: 3, deadLetterQueue: deadLetters },
  handle,
);

yield* Cloudflare.Queues.consumeQueueMessages<Job>(deadLetters, (messages) =>
  Stream.runForEach(messages, (message) =>
    Effect.logError(\`gave up on ${message.body.key} after ${message.attempts} attempts\`),
  ),
);
```

Replace the log with what the failure needs: store the message to replay later, or alert someone. A dead-letter queue owned by another stack can be passed as `yield* Cloudflare.Queues.Queue.ref("DeadLetters", { stack: "shared", stage: "production" })`, and one Alchemy doesn’t manage, by its name.

## Send a stream of messages

`QueueSink` turns a Queue into an Effect Sink. Run any `Stream` of message bodies into it, and it sends them in batches:

```typescript
const enriched = yield* Cloudflare.Queues.QueueSink(Enriched);

yield* Cloudflare.Queues.consumeQueueMessages<Click>(Clicks, (messages) =>
  messages.pipe(
    Stream.map((message) => ({ ...message.body, receivedAt: Date.now() })),
    Stream.run(enriched),
  ),
);
```

This Worker reads one Queue, enriches each message, and writes to another. Provide `Cloudflare.Queues.QueueSinkBinding`.

## Receive events from Cloudflare

Cloudflare can send its own events to a Queue: a Workflow instance completed, a Worker build failed, a bucket was created. A **subscription** connects an event source to a Queue:

```typescript
const events = yield* Cloudflare.Queues.Queue("WorkflowEvents");

yield* Cloudflare.Queues.Subscription("IngestionEvents", {
  source: worker.env.INGESTION,
  events: ["instance.completed", "instance.errored"],
  queueId: events.queueId,
});
```

The Queue still needs a consumer. These resources can be a source, directly or through `.ref(...)`:

| Resource | Events about |
| --- | --- |
| `Cloudflare.Workflows.WorkflowResource` | Instances of that Workflow |
| `Cloudflare.Worker` | Workers Builds runs for that Worker |
| `Cloudflare.AI.Model` | Batch inference for that model |
| `Cloudflare.KV.Namespace` | Every namespace in the account |
| `Cloudflare.R2.Bucket` | Every bucket in the account |
| `Cloudflare.R2.SuperSlurperJob` | Every migration job in the account |
| `Cloudflare.Vectorize.Index` | Every index in the account |
| `Cloudflare.Images.Variant` | Every image upload in the account |

The account-wide sources report on every resource of that kind, not just the one you pass. An account allows one subscription per source. Events can take a while to start arriving after a subscription is created, so check that one arrives before relying on it.
