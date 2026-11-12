---
url: https://alchemy.run/gcp/messaging/pubsub
title: "Pub/Sub"
description: "Publish to Pub/Sub topics from Cloud Run, consume them with push delivery or a pull loop, filter by attribute, and dead-letter failures."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Pub/Sub moves messages between services. A producer publishes to a topic. Each consumer gets its own subscription and sees every message it subscribes to, independent of the others.

## Topics and subscriptions

A **topic** is where messages are published. A **subscription** is one consumer’s own stream of them: every subscription gets every message published after it was created, unless its filter says otherwise, and acknowledges them separately. A subscription either **pushes** each message to a URL or waits for a consumer to **pull**.

<svg viewBox="0 0 776.2 242" style="max-width: 893px" role="img" aria-label="One publish reaches every subscription whose filter matches. Each acknowledges its own copy, so a slow Worker never holds back the Api."><defs><marker id="alc-fan-arrow-2desy6" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="283" y="8" width="269" height="226" rx="10"></rect><text fill="currentColor" x="293" y="26">Orders · topic</text></g> <rect stroke="currentColor" fill="none" x="8" y="81.5" width="223" height="79" rx="8"></rect><text fill="currentColor" x="20" y="101.5">message</text> <text fill="currentColor" x="20" y="121.5" xml:space="preserve">"data": { "orderId": "o_104" }</text> <text fill="currentColor" x="20" y="136.5" xml:space="preserve">"attributes":</text><text fill="currentColor" x="20" y="151.5" xml:space="preserve"> { "type": "order.created" }</text> <g><path stroke="currentColor" fill="none" d="M 231 121 C 262 121, 262 64, 293 64" marker-end="url(#alc-fan-arrow-2desy6)"></path><g><rect stroke="currentColor" fill="none" x="295" y="38" width="245" height="52" rx="7"></rect><text fill="currentColor" x="307" y="55">push subscription</text> <text fill="currentColor" x="307" y="72.5" xml:space="preserve">no filter</text> <line stroke-opacity="0.2" stroke="currentColor" x1="540" y1="64" x2="626" y2="64" marker-end="url(#alc-fan-arrow-2desy6)"></line></g><rect stroke="currentColor" fill="none" x="628" y="41" width="140.2" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="698.1" y="61" text-anchor="middle" opacity="1">Api</text> <text fill="currentColor" x="698.1" y="77" text-anchor="middle">Cloud Run service</text></g> <g><path stroke="currentColor" fill="none" d="M 231 121 C 262 121, 262 130, 293 130" marker-end="url(#alc-fan-arrow-2desy6)"></path><g><rect stroke="currentColor" fill="none" x="295" y="104" width="245" height="52" rx="7"></rect><text fill="currentColor" x="307" y="121">pull subscription</text> <text fill="currentColor" x="307" y="138.5" xml:space="preserve">no filter</text> <line stroke-opacity="0.2" stroke="currentColor" x1="540" y1="130" x2="626" y2="130" marker-end="url(#alc-fan-arrow-2desy6)"></line></g><rect stroke="currentColor" fill="none" x="628" y="107" width="140.2" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="698.1" y="127" text-anchor="middle" opacity="1">Worker</text> <text fill="currentColor" x="698.1" y="143" text-anchor="middle">worker pool</text></g> <g><path stroke="currentColor" fill="none" d="M 231 121 C 262 121, 262 196, 293 196"></path><g><rect stroke="currentColor" fill="none" x="295" y="170" width="245" height="52" rx="7"></rect><text fill="currentColor" x="307" y="187">push subscription</text> <text fill="currentColor" x="307" y="204.5" xml:space="preserve">attributes.type = "order.refunded"</text> <text fill="currentColor" x="552" y="200">no match</text></g> <rect stroke="currentColor" fill="none" x="628" y="173" width="140.2" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="698.1" y="193" text-anchor="middle" opacity="0.6">Refunds</text> <text fill="currentColor" x="698.1" y="209" text-anchor="middle">Cloud Run service</text></g></svg> <svg viewBox="0 0 318 561" style="max-width: 366px" role="img" aria-label="One publish reaches every subscription whose filter matches. Each acknowledges its own copy, so a slow Worker never holds back the Api."><defs><marker id="alc-fan-arrow-n-2desy6" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="14" y="93" width="304" height="466" rx="10"></rect><text fill="currentColor" x="24" y="111">Orders · topic</text></g> <rect stroke="currentColor" fill="none" x="0" y="0" width="223" height="79" rx="8"></rect><text fill="currentColor" x="12" y="20">message</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">"data": { "orderId": "o_104" }</text> <text fill="currentColor" x="12" y="55" xml:space="preserve">"attributes":</text><text fill="currentColor" x="12" y="70" xml:space="preserve"> { "type": "order.created" }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="79" x2="8" y2="449"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="157" x2="20" y2="157" marker-end="url(#alc-fan-arrow-n-2desy6)"></line><g><rect stroke="currentColor" fill="none" x="22" y="131" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="148">push subscription</text> <text fill="currentColor" x="34" y="165.5" xml:space="preserve">no filter</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="183" x2="166" y2="211" marker-end="url(#alc-fan-arrow-n-2desy6)"></line></g><rect stroke="currentColor" fill="none" x="95.9" y="213" width="140.2" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="233" text-anchor="middle" opacity="1">Api</text> <text fill="currentColor" x="166" y="249" text-anchor="middle">Cloud Run service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="303" x2="20" y2="303" marker-end="url(#alc-fan-arrow-n-2desy6)"></line><g><rect stroke="currentColor" fill="none" x="22" y="277" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="294">pull subscription</text> <text fill="currentColor" x="34" y="311.5" xml:space="preserve">no filter</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="329" x2="166" y2="357" marker-end="url(#alc-fan-arrow-n-2desy6)"></line></g><rect stroke="currentColor" fill="none" x="95.9" y="359" width="140.2" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="379" text-anchor="middle" opacity="1">Worker</text> <text fill="currentColor" x="166" y="395" text-anchor="middle">worker pool</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="449" x2="20" y2="449"></line><g><rect stroke="currentColor" fill="none" x="22" y="423" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="440">push subscription</text> <text fill="currentColor" x="34" y="457.5" xml:space="preserve">attributes.type = "order.refunded"</text> <text fill="currentColor" x="166" y="494" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="95.9" y="505" width="140.2" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="525" text-anchor="middle" opacity="0.6">Refunds</text> <text fill="currentColor" x="166" y="541" text-anchor="middle">Cloud Run service</text></g></svg>

One publish reaches every subscription whose filter matches. Each acknowledges its own copy, so a slow Worker never holds back the Api.

## Create a topic

```typescript
import * as GCP from "@alchemy.run/gcp";

export const Orders = GCP.PubSub.Topic("Orders", {});
```

## Publish

`WriteTopic` returns a client with `publish`:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Orders } from "./resources.ts";

export default class Api extends GCP.Function<Api>()(
  "Api",
  { main: import.meta.url, invokerIamDisabled: true },
  Effect.gen(function* () {
    const orders = yield* GCP.PubSub.WriteTopic(Orders);

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const order = yield* request.json;
        const messageId = yield* orders.publish({
          data: JSON.stringify(order),
          attributes: { type: "order.created" },
        });
        return yield* HttpServerResponse.json({ messageId }, { status: 202 });
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(GCP.PubSub.WriteTopicHttp)),
) {}
```

`publish` takes a string or bytes and returns the message id. Attributes can be filtered on without reading the body. The binding grants `roles/pubsub.publisher` on this topic only.

## Consume with push

A Cloud Run service receives messages over HTTP:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import * as Stream from "effect/Stream";
import { Orders } from "./resources.ts";

export default class Email extends GCP.Function<Email>()(
  "Email",
  { main: import.meta.url },
  Effect.gen(function* () {
    yield* GCP.PubSub.consumeTopicMessages(Orders, (messages) =>
      messages.pipe(
        Stream.runForEach(({ message }) =>
          Effect.gen(function* () {
            const order = JSON.parse(atob(message.data ?? ""));
            yield* Effect.log(\`confirm order ${order.id}\`);
          }),
        ),
        Effect.orDie,
      ),
    );

    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }).pipe(Effect.provide(GCP.Run.TopicEventSource)),
) {}
```

Alchemy creates a push subscription to a private path on the service, signed with the service’s own identity. The service stays private: only Pub/Sub’s signed requests get through. A success acks the message. A failure answers 500 and Pub/Sub delivers it again.

## Filter by attribute

Subscribe to a subset of messages. Pub/Sub applies the filter, so other messages never reach the service:

```typescript
yield* GCP.PubSub.consumeTopicMessages(
  Orders,
  { filter: 'attributes.type = "order.created"' },
  (messages) => messages.pipe(Stream.runForEach(handle), Effect.orDie),
);
```

## Dead-letter failures

Stop retrying a message after a few tries and park it on another topic:

```typescript
yield* GCP.PubSub.consumeTopicMessages(
  Orders,
  {
    deadLetter: { topic: yield* FailedOrders, maxDeliveryAttempts: 5 },
  },
  (messages) => messages.pipe(Stream.runForEach(handle), Effect.orDie),
);
```

<svg viewBox="-4 28 716.6 200" style="max-width: 824px" role="img" aria-label="A message leaves its subscription only when it's acknowledged. One that fails maxDeliveryAttempts times moves to the dead-letter topic."><defs><marker id="alc-sm-arrow-e7wnwn" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 180.2 63 Q 240.2 63 296.19999999999993 63" marker-end="url(#alc-sm-arrow-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 440.4 63 Q 500.4 63 556.4 63" marker-end="url(#alc-sm-arrow-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 338.09999999999997 86 Q 279.29999999999995 128 334.84506611517304 167.67504722512362" marker-end="url(#alc-sm-arrow-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 402.49999999999994 170 Q 461.29999999999995 128 405.75493388482687 88.32495277487638" marker-end="url(#alc-sm-arrow-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 440.4 193 Q 500.4 193 556.4 193" marker-end="url(#alc-sm-arrow-e7wnwn)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-e7wnwn)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="67" text-anchor="middle">outstanding</text></g> <g><rect stroke="currentColor" fill="none" x="300.19999999999993" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="370.29999999999995" y="61" text-anchor="middle">delivered</text> <text fill="currentColor" x="370.29999999999995" y="76" text-anchor="middle">to the handler</text></g> <g><rect stroke="currentColor" fill="none" x="560.4" y="40" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="564.4" y="44" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="630.5" y="61" text-anchor="middle">acknowledged</text> <text fill="currentColor" x="630.5" y="76" text-anchor="middle">removed</text></g> <g><rect stroke="currentColor" fill="none" x="300.19999999999993" y="170" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="370.29999999999995" y="191" text-anchor="middle">failed</text> <text fill="currentColor" x="370.29999999999995" y="206" text-anchor="middle">500 or deadline</text></g> <g><rect stroke="currentColor" fill="none" x="560.4" y="170" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="564.4" y="174" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="630.5" y="191" text-anchor="middle">FailedOrders</text> <text fill="currentColor" x="630.5" y="206" text-anchor="middle">dead-letter topic</text></g> <g><rect stroke="currentColor" fill="none" x="194.59999999999997" y="53" width="89.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="239.19999999999996" y="66" text-anchor="middle">push or pull</text></g> <g><rect stroke="currentColor" fill="none" x="441.59999999999997" y="53" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="499.4" y="66" text-anchor="middle">handler succeeds</text></g> <g><rect stroke="currentColor" fill="none" x="259.98626652879324" y="117.41876180628091" width="95.8" height="17" rx="4"></rect><text fill="currentColor" x="307.8862665287932" y="130.4187618062809" text-anchor="middle">handler fails</text></g> <g><rect stroke="currentColor" fill="none" x="391.4137334712067" y="118.58123819371909" width="82.6" height="17" rx="4"></rect><text fill="currentColor" x="432.7137334712067" y="131.5812381937191" text-anchor="middle">redelivered</text></g> <g><rect stroke="currentColor" fill="none" x="441.59999999999997" y="183" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="499.4" y="196" text-anchor="middle">after 5 attempts</text></g></svg> <svg viewBox="28 -4 410.20000000000005 378" style="max-width: 472px" role="img" aria-label="A message leaves its subscription only when it's acknowledged. One that fails maxDeliveryAttempts times moves to the dead-letter topic."><defs><marker id="alc-sm-arrow-n-e7wnwn" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 110.1 86 Q 110.1 132 110.1 174" marker-end="url(#alc-sm-arrow-n-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 110.1 224 Q 110.1 270 110.1 312" marker-end="url(#alc-sm-arrow-n-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 120.36785714285713 178 Q 233.10000000000002 -74.51999999999998 344.2015394956473 174.34744847024996" marker-end="url(#alc-sm-arrow-n-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 345.83214285714286 224 Q 233.10000000000002 476.52 121.99846050435269 227.65255152975004" marker-end="url(#alc-sm-arrow-n-e7wnwn)"></path><path stroke="currentColor" fill="none" d="M 356.1 224 Q 356.1 270 356.1 312" marker-end="url(#alc-sm-arrow-n-e7wnwn)"></path><g><g><circle stroke="currentColor" fill="none" cx="110.1" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="110.1" y1="20" x2="110.1" y2="36" marker-end="url(#alc-sm-arrow-n-e7wnwn)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="67" text-anchor="middle">outstanding</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="199" text-anchor="middle">delivered</text> <text fill="currentColor" x="110.1" y="214" text-anchor="middle">to the handler</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="316" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="44" y="320" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="110.1" y="337" text-anchor="middle">acknowledged</text> <text fill="currentColor" x="110.1" y="352" text-anchor="middle">removed</text></g> <g><rect stroke="currentColor" fill="none" x="286" y="178" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="356.1" y="199" text-anchor="middle">failed</text> <text fill="currentColor" x="356.1" y="214" text-anchor="middle">500 or deadline</text></g> <g><rect stroke="currentColor" fill="none" x="286" y="316" width="140.2" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="290" y="320" width="132.2" height="38" rx="19"></rect><text fill="currentColor" x="356.1" y="337" text-anchor="middle">FailedOrders</text> <text fill="currentColor" x="356.1" y="352" text-anchor="middle">dead-letter topic</text></g> <g><rect stroke="currentColor" fill="none" x="65.5" y="121" width="89.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="110.1" y="134" text-anchor="middle">push or pull</text></g> <g><rect stroke="currentColor" fill="none" x="52.3" y="259" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="110.1" y="272" text-anchor="middle">handler succeeds</text></g> <g><rect stroke="currentColor" fill="none" x="136.89234915962612" y="40.8268621175625" width="95.8" height="17" rx="4"></rect><text fill="currentColor" x="227.69234915962613" y="53.8268621175625" text-anchor="end">handler fails</text></g> <g><rect stroke="currentColor" fill="none" x="233.50765084037388" y="341.1731378824375" width="82.6" height="17" rx="4"></rect><text fill="currentColor" x="238.50765084037388" y="354.1731378824375" text-anchor="start">redelivered</text></g> <g><rect stroke="currentColor" fill="none" x="298.3" y="259" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="356.1" y="272" text-anchor="middle">after 5 attempts</text></g></svg>

A message leaves its subscription only when it's acknowledged. One that fails maxDeliveryAttempts times moves to the dead-letter topic.

A topic with no subscription drops what is published to it. Give the dead-letter topic a subscription so failed messages are kept:

```typescript
export const FailedOrdersInbox = Effect.gen(function* () {
  const topic = yield* FailedOrders;
  return yield* GCP.PubSub.Subscription("FailedOrdersInbox", {
    topic: topic.name,
    messageRetentionDuration: "604800s",
  });
});
```

## Consume with pull

A [worker pool](../compute/worker-pools.md) or a [job](../compute/jobs.md) has no URL. Pull instead:

```typescript
yield* GCP.PubSub.consumeTopicMessages(
  Orders,
  { maxMessages: 50 },
  (messages) => messages.pipe(Stream.runForEach(handle), Effect.orDie),
);
// …provided with Effect.provide(GCP.Run.TopicPullEventSource)
```

A batch is acked after your handler succeeds.

## Drain a subscription in batches

To control the batch yourself, bind a pull subscription with `ReadSubscription`. A [job](../compute/jobs.md) can drain it into BigQuery and exit:

```typescript
const subscription = yield* GCP.PubSub.ReadSubscription(Inbox);
const warehouse = yield* GCP.BigQuery.WriteTable(table);

return {
  run: Effect.gen(function* () {
    const received = yield* subscription
      .pull({ maxMessages: 500 })
      .pipe(Effect.timeoutOption("10 seconds"));
    const messages = Option.getOrElse(received, () => []);
    if (messages.length === 0) return 0;

    const rows = messages.map((message) => JSON.parse(message.text));
    yield* warehouse.insert(rows, { insertIds: rows.map((row) => row.id) });
    yield* subscription.acknowledge(messages.map((message) => message.ackId));
    return messages.length;
  }).pipe(
    Effect.repeat({ until: (count) => count === 0 }),
    Effect.asVoid,
    Effect.orDie,
  ),
};
```

Insert before you acknowledge. A crash in between delivers the batch again, and `insertIds` drops the duplicates. Acknowledging first would lose the events.

## Handle repeats

Pub/Sub delivers at least once. Make handlers safe to run twice: write with `set` keyed by an id, or use `insertIds` in BigQuery.

## Examples

- [`gcp-pubsub-fanout`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-pubsub-fanout) — one topic, two push consumers, an attribute filter, and a dead-letter topic.
- [`gcp-pubsub-worker`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-pubsub-worker) — a job queue on a worker pool.
- [`gcp-event-pipeline`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-event-pipeline) — a service publishes, a job drains into BigQuery.

## Where next

- [Cloud Scheduler](scheduler.md) — run the drain on a timer.
- [PubSub reference](https://alchemy.run/providers/gcp/pubsub#topic) — every prop and attribute.
