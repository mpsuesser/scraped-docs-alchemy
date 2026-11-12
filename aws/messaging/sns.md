---
url: https://alchemy.run/aws/messaging/sns
title: "SNS"
description: "How SNS Topics broadcast one message to every subscriber, how to publish from a runtime, subscribe Functions and Queues, and filter what each subscriber receives."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

SNS broadcasts messages. A publisher sends a message to a **Topic** once, and SNS pushes a copy to every **subscription**: a Lambda Function, an SQS Queue, an HTTPS endpoint, an email address. The publisher doesn’t know who’s listening, so adding a subscriber never changes the code that publishes.

## A Topic delivers a copy to everyone

A [Queue](sqs.md) hands each message to one consumer. A Topic hands each message to every subscriber, each with its own copy.

<svg viewBox="0 0 421 200" style="max-width: 484px" role="img" aria-label="Api publishes once, and each subscription gets a copy of its own. Deleting the copy in EmailJobs leaves the others untouched."><defs><marker id="alc-fan-arrow-cgqt72" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><rect stroke="currentColor" fill="none" x="8" y="68" width="145" height="64" rx="8"></rect><text fill="currentColor" x="20" y="88">message</text> <text fill="currentColor" x="20" y="108" xml:space="preserve">"orderId": "o_104"</text> <text fill="currentColor" x="20" y="123" xml:space="preserve">"total": 120</text> <g><path stroke="currentColor" fill="none" d="M 153 100 C 216 100, 216 34, 279 34" marker-end="url(#alc-fan-arrow-cgqt72)"></path><rect stroke="currentColor" fill="none" x="281" y="11" width="132" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="347" y="31" text-anchor="middle" opacity="1">EmailJobs</text> <text fill="currentColor" x="347" y="47" text-anchor="middle">SQS Queue</text></g> <g><path stroke="currentColor" fill="none" d="M 153 100 C 216 100, 216 100, 279 100" marker-end="url(#alc-fan-arrow-cgqt72)"></path><rect stroke="currentColor" fill="none" x="281" y="77" width="132" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="347" y="97" text-anchor="middle" opacity="1">AnalyticsJobs</text> <text fill="currentColor" x="347" y="113" text-anchor="middle">SQS Queue</text></g> <g><path stroke="currentColor" fill="none" d="M 153 100 C 216 100, 216 166, 279 166" marker-end="url(#alc-fan-arrow-cgqt72)"></path><rect stroke="currentColor" fill="none" x="281" y="143" width="132" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="347" y="163" text-anchor="middle" opacity="1">Audit</text> <text fill="currentColor" x="347" y="179" text-anchor="middle">Lambda Function</text></g></svg> <svg viewBox="0 0 318 268" style="max-width: 366px" role="img" aria-label="Api publishes once, and each subscription gets a copy of its own. Deleting the copy in EmailJobs leaves the others untouched."><defs><marker id="alc-fan-arrow-n-cgqt72" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><rect stroke="currentColor" fill="none" x="0" y="0" width="145" height="64" rx="8"></rect><text fill="currentColor" x="12" y="20">message</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">"orderId": "o_104"</text> <text fill="currentColor" x="12" y="55" xml:space="preserve">"total": 120</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="64" x2="8" y2="241"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="113" x2="20" y2="113" marker-end="url(#alc-fan-arrow-n-cgqt72)"></line><rect stroke="currentColor" fill="none" x="104" y="90" width="132" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="170" y="110" text-anchor="middle" opacity="1">EmailJobs</text> <text fill="currentColor" x="170" y="126" text-anchor="middle">SQS Queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="177" x2="20" y2="177" marker-end="url(#alc-fan-arrow-n-cgqt72)"></line><rect stroke="currentColor" fill="none" x="104" y="154" width="132" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="170" y="174" text-anchor="middle" opacity="1">AnalyticsJobs</text> <text fill="currentColor" x="170" y="190" text-anchor="middle">SQS Queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="241" x2="20" y2="241" marker-end="url(#alc-fan-arrow-n-cgqt72)"></line><rect stroke="currentColor" fill="none" x="104" y="218" width="132" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="170" y="238" text-anchor="middle" opacity="1">Audit</text> <text fill="currentColor" x="170" y="254" text-anchor="middle">Lambda Function</text></g></svg>

Api publishes once, and each subscription gets a copy of its own. Deleting the copy in EmailJobs leaves the others untouched.

SNS pushes each copy as soon as it arrives and retries deliveries that fail. It doesn’t store messages for later, so a subscriber that needs to work at its own pace, or survive being down for a while, subscribes through a Queue.

## Declare a Topic

```typescript
import * as AWS from "@alchemy.run/aws";

export const Orders = AWS.SNS.Topic("Orders");
```

For messages that must arrive in order and only once, make it a FIFO Topic, and subscribe FIFO Queues to it:

```typescript
export const Orders = AWS.SNS.Topic("Orders", {
  fifo: true,
  attributes: { ContentBasedDeduplication: "true" },
});
```

Changing `fifo` replaces the Topic. Everything else updates in place.

## Publish

Bind `SNS.Publish` in the publisher. Its role gets `sns:Publish` on this Topic only:

```typescript
const publish = yield* AWS.SNS.Publish(Orders);
yield* publish({ Message: JSON.stringify({ orderId }), Subject: "OrderCreated" });
```

Provide `AWS.SNS.PublishHttp` to the runtime. To publish up to ten messages per call, bind `SNS.PublishBatch`. To publish a whole `Stream`, bind `SNS.TopicSink`, which batches it for you, and provide `AWS.SNS.TopicSinkHttp` with `AWS.SNS.PublishBatchHttp` beneath it.

A Lambda Function subscribes from Construction and receives each notification in a `Stream`:

```typescript
export default class Audit extends AWS.Lambda.Function<Audit>()(
  "Audit",
  { main: import.meta.url, functionUrl: false },
  Effect.gen(function* () {
    const orders = yield* Orders;
    yield* AWS.SNS.consumeTopicNotifications(orders, (stream) =>
      stream.pipe(Stream.runForEach((n) => Console.log(\`${n.Subject}: ${n.Message}\`))),
    );
    return {};
  }).pipe(Effect.provide(AWS.Lambda.TopicEventSource)),
) {}
```

Deploy creates the subscription and the Lambda permission that lets SNS invoke the Function. Each notification carries the `Message`, `Subject`, `MessageId` and `TopicArn`.

## Fan out to Queues

The usual shape is one Queue per subscriber. Each Queue gets its own copy and is drained by its own consumer, at its own pace, with its own retries and dead-letter queue. A subscription is a resource of its own:

```typescript
const orders = yield* Orders;
const emails = yield* AWS.SQS.Queue("EmailJobs");

yield* AWS.SNS.Subscription("EmailSubscription", {
  topicArn: orders.topicArn,
  protocol: "sqs",
  endpoint: emails.queueArn,
  attributes: { RawMessageDelivery: "true" },
});
```

SNS needs permission to send to the Queue. Give it in the Queue’s own policy, limited to this Topic:

```typescript
const emails = yield* AWS.SQS.Queue("EmailJobs", {
  policy: {
    Version: "2012-10-17",
    Statement: [
      {
        Effect: "Allow",
        Principal: { Service: "sns.amazonaws.com" },
        Action: "sqs:SendMessage",
        Resource: "*",
        Condition: { ArnEquals: { "aws:SourceArn": orders.topicArn } },
      },
    ],
  },
});
```

`RawMessageDelivery` delivers the message body as is. Without it, each message arrives wrapped in a JSON envelope with the subject and Topic. From here, each Queue is an ordinary [SQS consumer](sqs.md#consume-messages).

## Filter what each subscriber sees

A subscriber rarely wants every message. A **filter policy** tells SNS which ones to deliver, matched on message attributes, so the rest never invoke the subscriber at all:

```typescript
yield* AWS.SNS.consumeTopicNotifications(
  orders,
  { attributes: { FilterPolicy: JSON.stringify({ type: ["order.created"] }) } },
  (stream) => stream.pipe(Stream.runForEach((n) => Console.log(n.Message))),
);
```

<svg viewBox="0 0 730.5 200" style="max-width: 840px" role="img" aria-label="SNS checks each subscription's filter policy before it delivers. Refunds never sees this message, and is never invoked for it."><defs><marker id="alc-fan-arrow-7dhw54" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><rect stroke="currentColor" fill="none" x="8" y="60.5" width="216.5" height="79" rx="8"></rect><text fill="currentColor" x="20" y="80.5">message</text> <text fill="currentColor" x="20" y="100.5" xml:space="preserve">"Message": "…"</text> <text fill="currentColor" x="20" y="115.5" xml:space="preserve">"MessageAttributes":</text><text fill="currentColor" x="20" y="130.5" xml:space="preserve"> { "type": "order.created" }</text> <g><path stroke="currentColor" fill="none" d="M 224.5 100 C 255.5 100, 255.5 34, 286.5 34" marker-end="url(#alc-fan-arrow-7dhw54)"></path><g><rect stroke="currentColor" fill="none" x="288.5" y="8" width="219" height="52" rx="7"></rect><text fill="currentColor" x="300.5" y="25">no filter policy</text> <text fill="currentColor" x="300.5" y="42.5" xml:space="preserve">every message</text> <line stroke-opacity="0.2" stroke="currentColor" x1="507.5" y1="34" x2="593.5" y2="34" marker-end="url(#alc-fan-arrow-7dhw54)"></line></g><rect stroke="currentColor" fill="none" x="595.5" y="11" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="659" y="31" text-anchor="middle" opacity="1">EmailJobs</text> <text fill="currentColor" x="659" y="47" text-anchor="middle">SQS Queue</text></g> <g><path stroke="currentColor" fill="none" d="M 224.5 100 C 255.5 100, 255.5 100, 286.5 100" marker-end="url(#alc-fan-arrow-7dhw54)"></path><g><rect stroke="currentColor" fill="none" x="288.5" y="74" width="219" height="52" rx="7"></rect><text fill="currentColor" x="300.5" y="91">filter policy</text> <text fill="currentColor" x="300.5" y="108.5" xml:space="preserve">{ "type": ["order.created"] }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="507.5" y1="100" x2="593.5" y2="100" marker-end="url(#alc-fan-arrow-7dhw54)"></line></g><rect stroke="currentColor" fill="none" x="595.5" y="77" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="659" y="97" text-anchor="middle" opacity="1">Audit</text> <text fill="currentColor" x="659" y="113" text-anchor="middle">Lambda Function</text></g> <g><path stroke="currentColor" fill="none" d="M 224.5 100 C 255.5 100, 255.5 166, 286.5 166"></path><g><rect stroke="currentColor" fill="none" x="288.5" y="140" width="219" height="52" rx="7"></rect><text fill="currentColor" x="300.5" y="157">filter policy</text> <text fill="currentColor" x="300.5" y="174.5" xml:space="preserve">{ "type": ["order.refunded"] }</text> <text fill="currentColor" x="519.5" y="170">no match</text></g> <rect stroke="currentColor" fill="none" x="595.5" y="143" width="127" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="659" y="163" text-anchor="middle" opacity="0.6">Refunds</text> <text fill="currentColor" x="659" y="179" text-anchor="middle">Lambda Function</text></g></svg> <svg viewBox="0 0 318 529" style="max-width: 366px" role="img" aria-label="SNS checks each subscription's filter policy before it delivers. Refunds never sees this message, and is never invoked for it."><defs><marker id="alc-fan-arrow-n-7dhw54" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><rect stroke="currentColor" fill="none" x="0" y="0" width="216.5" height="79" rx="8"></rect><text fill="currentColor" x="12" y="20">message</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">"Message": "…"</text> <text fill="currentColor" x="12" y="55" xml:space="preserve">"MessageAttributes":</text><text fill="currentColor" x="12" y="70" xml:space="preserve"> { "type": "order.created" }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="79" x2="8" y2="423"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="131" x2="20" y2="131" marker-end="url(#alc-fan-arrow-n-7dhw54)"></line><g><rect stroke="currentColor" fill="none" x="22" y="105" width="296" height="52" rx="7"></rect><text fill="currentColor" x="34" y="122">no filter policy</text> <text fill="currentColor" x="34" y="139.5" xml:space="preserve">every message</text> <line stroke-opacity="0.2" stroke="currentColor" x1="170" y1="157" x2="170" y2="185" marker-end="url(#alc-fan-arrow-n-7dhw54)"></line></g><rect stroke="currentColor" fill="none" x="106.5" y="187" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="170" y="207" text-anchor="middle" opacity="1">EmailJobs</text> <text fill="currentColor" x="170" y="223" text-anchor="middle">SQS Queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="277" x2="20" y2="277" marker-end="url(#alc-fan-arrow-n-7dhw54)"></line><g><rect stroke="currentColor" fill="none" x="22" y="251" width="296" height="52" rx="7"></rect><text fill="currentColor" x="34" y="268">filter policy</text> <text fill="currentColor" x="34" y="285.5" xml:space="preserve">{ "type": ["order.created"] }</text> <line stroke-opacity="0.2" stroke="currentColor" x1="170" y1="303" x2="170" y2="331" marker-end="url(#alc-fan-arrow-n-7dhw54)"></line></g><rect stroke="currentColor" fill="none" x="106.5" y="333" width="127" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="170" y="353" text-anchor="middle" opacity="1">Audit</text> <text fill="currentColor" x="170" y="369" text-anchor="middle">Lambda Function</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="423" x2="20" y2="423"></line><g><rect stroke="currentColor" fill="none" x="22" y="397" width="296" height="52" rx="7"></rect><text fill="currentColor" x="34" y="414">filter policy</text> <text fill="currentColor" x="34" y="431.5" xml:space="preserve">{ "type": ["order.refunded"] }</text> <text fill="currentColor" x="170" y="468" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="106.5" y="479" width="127" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="170" y="499" text-anchor="middle" opacity="0.6">Refunds</text> <text fill="currentColor" x="170" y="515" text-anchor="middle">Lambda Function</text></g></svg>

SNS checks each subscription's filter policy before it delivers. Refunds never sees this message, and is never invoked for it.

Publishers set the attributes to match on with `MessageAttributes`. A `Subscription` takes the same `FilterPolicy` in its `attributes`.

## SNS, SQS or EventBridge

All three move messages between parts of an app, and each fits a different shape.

Use an [SQS Queue](sqs.md) when one consumer should do each piece of work, at its own pace, with retries. Use an SNS Topic when several independent subscribers should each react to the same message. Use [EventBridge](eventbridge.md) when many kinds of events share one bus and rules route each one by its content, including events from AWS services themselves and schedules.
