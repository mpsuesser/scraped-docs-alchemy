---
url: https://alchemy.run/aws/messaging/sqs
title: "SQS"
description: "How SQS Queues hand work from producers to consumers, how visibility timeouts, retries and dead-letter queues keep messages from being lost, and how runtimes send to and consume from a Queue."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

SQS is a message queue. A **producer** puts a message on a Queue and moves on. A **consumer** takes messages off later, does the work, and deletes them. The two never wait for each other, so a slow job doesn’t slow down the request that asked for it, and a burst of work waits in the Queue instead of overwhelming whatever does it.

## Consumers take messages in any order

A standard Queue doesn’t keep messages in a line. It spreads them across many servers, and each receive returns whichever messages it finds, up to 10 at a time. So consumers get messages roughly, but not exactly, in the order they were sent:

<svg viewBox="0 0 666.6 170" style="max-width: 767px" role="img" aria-label="Worker 1 has job 5 and Worker 2 has job 2, while job 1 still waits. The messages they hold stay in the Queue, hidden from everyone else."><defs><marker id="alc-q-arrow-esqyit" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="8" y="72" width="100" height="46" rx="8"></rect><text fill="currentColor" x="58" y="92" text-anchor="middle">Api</text> <text fill="currentColor" x="58" y="108" text-anchor="middle">producer</text> <line stroke-opacity="0.2" stroke="currentColor" x1="108" y1="95" x2="202.6" y2="95" marker-end="url(#alc-q-arrow-esqyit)"></line><text fill="currentColor" x="156.3" y="87" text-anchor="middle">SendMessage</text></g> <rect stroke="currentColor" fill="none" x="204.6" y="30" width="252" height="130" rx="10"></rect><text fill="currentColor" x="208.6" y="20">Jobs · SQS Queue</text> <g><rect stroke="currentColor" fill="none" x="220.6" y="44" width="64" height="30" rx="5"></rect><text fill="currentColor" x="252.6" y="63" text-anchor="middle" opacity="1">job 3</text></g> <g><rect stroke="currentColor" fill="none" x="220.6" y="104" width="64" height="30" rx="5"></rect><text fill="currentColor" x="252.6" y="123" text-anchor="middle" opacity="1">job 6</text></g> <g><rect stroke="currentColor" fill="none" x="298.6" y="58" width="64" height="30" rx="5"></rect><text fill="currentColor" x="330.6" y="77" text-anchor="middle" opacity="1">job 1</text></g> <g><rect stroke="currentColor" fill="none" x="298.6" y="97" width="64" height="30" rx="5"></rect><text fill="currentColor" x="330.6" y="116" text-anchor="middle" opacity="1">job 7</text></g> <g><path stroke="currentColor" fill="none" d="M 440.6 65 C 506.6 65, 496.6 53, 544.6 53" marker-end="url(#alc-q-arrow-esqyit)"></path><rect stroke="currentColor" fill="none" x="376.6" y="50" width="64" height="30" rx="5"></rect><text fill="currentColor" x="408.6" y="69" text-anchor="middle" opacity="0.75">job 5</text> <text fill="currentColor" x="408.6" y="92" text-anchor="middle">hidden</text></g> <g><path stroke="currentColor" fill="none" d="M 440.6 116 C 506.6 116, 496.6 137, 544.6 137" marker-end="url(#alc-q-arrow-esqyit)"></path><rect stroke="currentColor" fill="none" x="376.6" y="101" width="64" height="30" rx="5"></rect><text fill="currentColor" x="408.6" y="120" text-anchor="middle" opacity="0.75">job 2</text> <text fill="currentColor" x="408.6" y="143" text-anchor="middle">hidden</text></g> <g><rect stroke="currentColor" fill="none" x="546.6" y="30" width="112" height="46" rx="8"></rect><text fill="currentColor" x="602.6" y="50" text-anchor="middle">Worker 1</text> <text fill="currentColor" x="602.6" y="66" text-anchor="middle">consumer</text></g> <g><rect stroke="currentColor" fill="none" x="546.6" y="114" width="112" height="46" rx="8"></rect><text fill="currentColor" x="602.6" y="134" text-anchor="middle">Worker 2</text> <text fill="currentColor" x="602.6" y="150" text-anchor="middle">consumer</text></g></svg> <svg viewBox="0 0 318 322" style="max-width: 366px" role="img" aria-label="Worker 1 has job 5 and Worker 2 has job 2, while job 1 still waits. The messages they hold stay in the Queue, hidden from everyone else."><defs><marker id="alc-q-arrow-n-esqyit" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="109" y="0" width="100" height="46" rx="8"></rect><text fill="currentColor" x="159" y="20" text-anchor="middle">Api</text> <text fill="currentColor" x="159" y="36" text-anchor="middle">producer</text> <line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="46" x2="159" y2="68" marker-end="url(#alc-q-arrow-n-esqyit)"></line><text fill="currentColor" x="167" y="64">SendMessage</text></g> <text fill="currentColor" x="4" y="82">Jobs · SQS Queue</text> <rect stroke="currentColor" fill="none" x="0" y="90" width="318" height="124" rx="10"></rect><path stroke="currentColor" fill="none" d="M 44 188 V 220 C 44 244, 98 234, 98 268" marker-end="url(#alc-q-arrow-n-esqyit)"></path><path stroke="currentColor" fill="none" d="M 118 188 V 220 C 118 244, 220 234, 220 268" marker-end="url(#alc-q-arrow-n-esqyit)"></path><g><rect stroke="currentColor" fill="none" x="12" y="102" width="64" height="30" rx="5"></rect><text fill="currentColor" x="44" y="121" text-anchor="middle" opacity="1">job 3</text></g> <g><rect stroke="currentColor" fill="none" x="86" y="102" width="64" height="30" rx="5"></rect><text fill="currentColor" x="118" y="121" text-anchor="middle" opacity="1">job 6</text></g> <g><rect stroke="currentColor" fill="none" x="160" y="102" width="64" height="30" rx="5"></rect><text fill="currentColor" x="192" y="121" text-anchor="middle" opacity="1">job 1</text></g> <g><rect stroke="currentColor" fill="none" x="234" y="102" width="64" height="30" rx="5"></rect><text fill="currentColor" x="266" y="121" text-anchor="middle" opacity="1">job 7</text></g> <g><rect stroke="currentColor" x="12" y="158" width="64" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="44" y="177" text-anchor="middle" opacity="0.75">job 5</text> <text fill="currentColor" x="44" y="200" text-anchor="middle">hidden</text></g> <g><rect stroke="currentColor" x="86" y="158" width="64" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="118" y="177" text-anchor="middle" opacity="0.75">job 2</text> <text fill="currentColor" x="118" y="200" text-anchor="middle">hidden</text></g> <g><rect stroke="currentColor" fill="none" x="42" y="270" width="112" height="46" rx="8"></rect><text fill="currentColor" x="98" y="290" text-anchor="middle">Worker 1</text> <text fill="currentColor" x="98" y="306" text-anchor="middle">consumer</text></g> <g><rect stroke="currentColor" fill="none" x="164" y="270" width="112" height="46" rx="8"></rect><text fill="currentColor" x="220" y="290" text-anchor="middle">Worker 2</text> <text fill="currentColor" x="220" y="306" text-anchor="middle">consumer</text></g></svg>

Worker 1 has job 5 and Worker 2 has job 2, while job 1 still waits. The messages they hold stay in the Queue, hidden from everyone else.

## A message is received, then deleted

A message stays on the Queue until a consumer deletes it. Receiving it doesn’t remove it. It only hides it from other consumers for the **visibility timeout**, 30 seconds by default.

<svg viewBox="0 0 668 361" style="max-width: 768px" role="img" aria-label="Receiving a message hides it. Deleting it removes it. A message nobody deletes comes back, so the same job can run twice."><defs><marker id="alc-seq-arrow-zvyai1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-zvyai1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="357"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="36" text-anchor="middle">Api</text> <text fill="currentColor" x="82" y="51" text-anchor="middle">producer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="357"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">Jobs</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">SQS Queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="357"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="36" text-anchor="middle">Worker</text> <text fill="currentColor" x="418" y="51" text-anchor="middle">consumer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="586" y1="59" x2="586" y2="357"></line><rect stroke="currentColor" fill="none" x="520" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="586" y="36" text-anchor="middle">Worker</text> <text fill="currentColor" x="586" y="51" text-anchor="middle">another consumer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="85" x2="244" y2="85" marker-end="url(#alc-seq-arrow-zvyai1)"></line><rect stroke="currentColor" fill="none" x="125.69999999999999" y="66" width="80.6" height="15" rx="3"></rect><text fill="currentColor" x="166" y="78" text-anchor="middle">SendMessage</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="125" x2="412" y2="125" marker-end="url(#alc-seq-arrow-zvyai1)"></line><rect stroke="currentColor" fill="none" x="283.8" y="106" width="100.39999999999999" height="15" rx="3"></rect><text fill="currentColor" x="334" y="118" text-anchor="middle">ReceiveMessage</text></g> <g><rect stroke="currentColor" x="122.5" y="153" width="255" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="169" text-anchor="middle">hidden from other consumers for 30s</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="201" x2="256" y2="201" marker-end="url(#alc-seq-arrow-zvyai1)"></line><rect stroke="currentColor" fill="none" x="287.1" y="182" width="93.8" height="15" rx="3"></rect><text fill="currentColor" x="334" y="194" text-anchor="middle">DeleteMessage</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="16" y1="249" x2="652" y2="249"></line><rect stroke="currentColor" fill="none" x="16" y="240" width="219.70000000000002" height="18" rx="4"></rect><text fill="currentColor" x="22" y="253">or, if the Worker crashes first</text></g> <g><rect stroke="currentColor" x="162.10000000000002" y="269" width="175.79999999999998" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="285" text-anchor="middle">30s pass: visible again</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="317" x2="580" y2="317" marker-end="url(#alc-seq-arrow-warn-zvyai1)"></line><rect stroke="currentColor" fill="none" x="367.8" y="298" width="100.39999999999999" height="15" rx="3"></rect><text fill="currentColor" x="418" y="310" text-anchor="middle">ReceiveMessage</text></g></svg> <svg viewBox="-2 0 324 360" style="max-width: 373px" role="img" aria-label="Receiving a message hides it. Deleting it removes it. A message nobody deletes comes back, so the same job can run twice."><defs><marker id="alc-seq-arrow-n-zvyai1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-zvyai1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="33" y1="58" x2="33" y2="356"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="62" height="56" rx="8"></rect><text fill="currentColor" x="33" y="22" text-anchor="middle">Api</text> <text fill="currentColor" x="33" y="37" text-anchor="middle">producer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="118.33333333333333" y1="58" x2="118.33333333333333" y2="356"></line><rect stroke="currentColor" fill="none" x="87.33333333333333" y="2" width="62" height="56" rx="8"></rect><text fill="currentColor" x="118.33333333333333" y="22" text-anchor="middle">Jobs</text> <text fill="currentColor" x="118.33333333333333" y="37" text-anchor="middle">SQS</text> <text fill="currentColor" x="118.33333333333333" y="50" text-anchor="middle">Queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="203.66666666666666" y1="58" x2="203.66666666666666" y2="356"></line><rect stroke="currentColor" fill="none" x="172.66666666666666" y="2" width="62" height="56" rx="8"></rect><text fill="currentColor" x="203.66666666666666" y="22" text-anchor="middle">Worker</text> <text fill="currentColor" x="203.66666666666666" y="37" text-anchor="middle">consumer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="289" y1="58" x2="289" y2="356"></line><rect stroke="currentColor" fill="none" x="258" y="2" width="62" height="56" rx="8"></rect><text fill="currentColor" x="289" y="22" text-anchor="middle">Worker</text> <text fill="currentColor" x="289" y="37" text-anchor="middle">another</text> <text fill="currentColor" x="289" y="50" text-anchor="middle">consumer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="33" y1="84" x2="112.33333333333333" y2="84" marker-end="url(#alc-seq-arrow-n-zvyai1)"></line><rect stroke="currentColor" fill="none" x="35.36666666666666" y="65" width="80.6" height="15" rx="3"></rect><text fill="currentColor" x="75.66666666666666" y="77" text-anchor="middle">SendMessage</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="118.33333333333333" y1="124" x2="197.66666666666666" y2="124" marker-end="url(#alc-seq-arrow-n-zvyai1)"></line><rect stroke="currentColor" fill="none" x="110.80000000000001" y="105" width="100.39999999999999" height="15" rx="3"></rect><text fill="currentColor" x="161" y="117" text-anchor="middle">ReceiveMessage</text></g> <g><rect stroke="currentColor" x="2" y="152" width="255" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="129.5" y="168" text-anchor="middle">hidden from other consumers for 30s</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="203.66666666666666" y1="200" x2="124.33333333333333" y2="200" marker-end="url(#alc-seq-arrow-n-zvyai1)"></line><rect stroke="currentColor" fill="none" x="114.1" y="181" width="93.8" height="15" rx="3"></rect><text fill="currentColor" x="161" y="193" text-anchor="middle">DeleteMessage</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="248" x2="320" y2="248"></line><rect stroke="currentColor" fill="none" x="2" y="239" width="219.70000000000002" height="18" rx="4"></rect><text fill="currentColor" x="8" y="252">or, if the Worker crashes first</text></g> <g><rect stroke="currentColor" x="30.433333333333337" y="268" width="175.79999999999998" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="118.33333333333333" y="284" text-anchor="middle">30s pass: visible again</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="118.33333333333333" y1="316" x2="283" y2="316" marker-end="url(#alc-seq-arrow-warn-n-zvyai1)"></line><rect stroke="currentColor" fill="none" x="153.46666666666667" y="297" width="100.39999999999999" height="15" rx="3"></rect><text fill="currentColor" x="203.66666666666666" y="309" text-anchor="middle">ReceiveMessage</text></g></svg>

Receiving a message hides it. Deleting it removes it. A message nobody deletes comes back, so the same job can run twice.

If the consumer finishes, it deletes the message and it’s gone. If the consumer crashes or times out, the message becomes visible again and another consumer receives it. So a message is never lost to a failure, but it can be delivered more than once. Write consumers so that doing a job twice is safe: check whether it’s done already, or make the work idempotent.

## Declare a Queue

A Queue is a declaration. Put it in its own module, so the producer and the consumer can both import it:

```typescript
import * as AWS from "@alchemy.run/aws";

export const Jobs = AWS.SQS.Queue("Jobs");
```

Messages are kept for four days by default. Set `messageRetentionPeriod` to keep them longer, up to 14 days, and set `visibilityTimeout` longer than your consumer takes to finish one batch:

```typescript
export const Jobs = AWS.SQS.Queue("Jobs", { visibilityTimeout: "2 minutes" });
```

## Send messages

Bind `SQS.SendMessage` in the producer. Its role gets `sqs:SendMessage` on this Queue only:

```typescript
const sendMessage = yield* AWS.SQS.SendMessage(Jobs);
```

A message body is a string, up to 256 KB. Send JSON:

```typescript
yield* sendMessage({ MessageBody: JSON.stringify({ type: "thumbnail", key }) });
```

Provide `AWS.SQS.SendMessageHttp` to the producer. Sending returns as soon as SQS has stored the message, so the request handler can respond while the work waits.

## Consume messages

A Lambda Function consumes a Queue by subscribing to it from Construction. Lambda polls the Queue for you and invokes the Function with batches of messages:

```typescript
export default class Worker extends AWS.Lambda.Function<Worker>()(
  "Worker",
  { main: import.meta.url, functionUrl: false },
  Effect.gen(function* () {
    yield* AWS.SQS.consumeQueueMessages(Jobs, { batchSize: 10 }, (event) =>
      Effect.forEach(event.Records, (record) => Console.log(record.body), { discard: true }),
    );
    return {};
  }).pipe(Effect.provide(AWS.Lambda.QueueEventSource)),
) {}
```

Deploy creates the event source mapping and grants the Function `sqs:ReceiveMessage`, `sqs:DeleteMessage` and `sqs:GetQueueAttributes` on the Queue. When the handler succeeds, Lambda deletes the whole batch. When it fails, the whole batch becomes visible again and is retried.

A process on ECS or EC2 consumes the same way, with its own event source layer: it long-polls the Queue in a loop instead of being invoked.

## Retry only the messages that failed

Failing a whole batch for one bad message repeats the work that succeeded. Instead, return the ids of the messages that failed. Lambda deletes the rest and redelivers only those:

```typescript
yield* AWS.SQS.consumeQueueMessages(Jobs, { batchSize: 10 }, (event) =>
  Effect.filter(
    event.Records,
    (record) =>
      handleJob(record.body).pipe(
        Effect.as(false),
        Effect.catchCause(() => Effect.succeed(true)),
      ),
    { concurrency: 4 },
  ).pipe(
    Effect.map((failed) => ({
      batchItemFailures: failed.map((record) => ({ itemIdentifier: record.messageId })),
    })),
  ),
);
```

Messages in a batch are processed four at a time here, and one failure no longer drags the others back onto the Queue.

## Move poison messages aside

Some messages fail every time: malformed input, a bug, a record that no longer exists. Without a limit, they’re retried until they expire. A **dead-letter queue** catches them instead. After a message has been received `maxReceiveCount` times, SQS moves it there:

```typescript
const deadLetters = yield* AWS.SQS.Queue("JobsDeadLetters", { messageRetentionPeriod: "14 days" });
const jobs = yield* AWS.SQS.Queue("Jobs", {
  redrivePolicy: { deadLetterTargetArn: deadLetters.queueArn, maxReceiveCount: 5 },
});
```

The work stops failing in a loop, and the failed messages wait for you to look at them. Put an [alarm](../observability/cloudwatch.md) on the dead-letter queue’s depth so you know when one arrives.

## Keep order with a FIFO queue

A standard Queue delivers messages roughly in order, at least once. A **FIFO** queue delivers them exactly in order and exactly once, within each **message group**:

```typescript
export const Ledger = AWS.SQS.Queue("Ledger", { fifo: true, contentBasedDeduplication: true });
```

Every message names its group, like an account id. Messages in one group are delivered one after another, and different groups are processed in parallel:

```typescript
yield* sendMessage({ MessageBody: body, MessageGroupId: accountId });
```

<svg viewBox="0 0 538.8 168" style="max-width: 620px" role="img" aria-label="Each group is a line of its own. While ana #1 is in flight, SQS hands out nothing else from acct_ana, so her entries run in order. acct_sam moves at the same time, on Worker 2."><defs><marker id="alc-q-arrow-jj31jh" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><rect stroke="currentColor" fill="none" x="8" y="30" width="320.8" height="128" rx="10"></rect><text fill="currentColor" x="12" y="20">Ledger · FIFO Queue</text> <text fill="currentColor" x="24" y="61">acct_ana</text> <text fill="currentColor" x="24" y="117">acct_sam</text> <g><path stroke="currentColor" fill="none" d="M 312.8 57 C 378.8 57, 368.8 53, 416.8 53" marker-end="url(#alc-q-arrow-jj31jh)"></path><rect stroke="currentColor" fill="none" x="248.8" y="42" width="64" height="30" rx="5"></rect><text fill="currentColor" x="280.8" y="61" text-anchor="middle" opacity="0.75">ana #1</text> <text fill="currentColor" x="280.8" y="84" text-anchor="middle">in flight</text></g> <g><rect stroke="currentColor" fill="none" x="170.8" y="42" width="64" height="30" rx="5"></rect><text fill="currentColor" x="202.8" y="61" text-anchor="middle" opacity="1">ana #2</text></g> <g><rect stroke="currentColor" fill="none" x="92.80000000000001" y="42" width="64" height="30" rx="5"></rect><text fill="currentColor" x="124.80000000000001" y="61" text-anchor="middle" opacity="1">ana #3</text></g> <g><path stroke="currentColor" fill="none" d="M 312.8 113 C 378.8 113, 368.8 135, 416.8 135" marker-end="url(#alc-q-arrow-jj31jh)"></path><rect stroke="currentColor" fill="none" x="248.8" y="98" width="64" height="30" rx="5"></rect><text fill="currentColor" x="280.8" y="117" text-anchor="middle" opacity="0.75">sam #1</text> <text fill="currentColor" x="280.8" y="140" text-anchor="middle">in flight</text></g> <g><rect stroke="currentColor" fill="none" x="170.8" y="98" width="64" height="30" rx="5"></rect><text fill="currentColor" x="202.8" y="117" text-anchor="middle" opacity="1">sam #2</text></g> <g><rect stroke="currentColor" fill="none" x="418.8" y="30" width="112" height="46" rx="8"></rect><text fill="currentColor" x="474.8" y="50" text-anchor="middle">Worker 1</text> <text fill="currentColor" x="474.8" y="66" text-anchor="middle">consumer</text></g> <g><rect stroke="currentColor" fill="none" x="418.8" y="112" width="112" height="46" rx="8"></rect><text fill="currentColor" x="474.8" y="132" text-anchor="middle">Worker 2</text> <text fill="currentColor" x="474.8" y="148" text-anchor="middle">consumer</text></g></svg> <svg viewBox="0 0 318 254" style="max-width: 366px" role="img" aria-label="Each group is a line of its own. While ana #1 is in flight, SQS hands out nothing else from acct_ana, so her entries run in order. acct_sam moves at the same time, on Worker 2."><defs><marker id="alc-q-arrow-n-jj31jh" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><text fill="currentColor" x="4" y="14">Ledger · FIFO Queue</text> <rect stroke="currentColor" fill="none" x="0" y="22" width="318" height="124" rx="10"></rect><text fill="currentColor" x="12" y="53">acct_ana</text> <text fill="currentColor" x="12" y="109">acct_sam</text> <path stroke="currentColor" fill="none" d="M 274 64 V 152 C 274 176, 98 166, 98 200" marker-end="url(#alc-q-arrow-n-jj31jh)"></path><path stroke="currentColor" fill="none" d="M 274 120 V 152 C 274 176, 220 166, 220 200" marker-end="url(#alc-q-arrow-n-jj31jh)"></path><g><rect stroke="currentColor" x="242" y="34" width="64" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="274" y="53" text-anchor="middle" opacity="0.75">ana #1</text> <text fill="currentColor" x="274" y="76" text-anchor="middle">in flight</text></g> <g><rect stroke="currentColor" fill="none" x="168" y="34" width="64" height="30" rx="5"></rect><text fill="currentColor" x="200" y="53" text-anchor="middle" opacity="1">ana #2</text></g> <g><rect stroke="currentColor" fill="none" x="94" y="34" width="64" height="30" rx="5"></rect><text fill="currentColor" x="126" y="53" text-anchor="middle" opacity="1">ana #3</text></g> <g><rect stroke="currentColor" x="242" y="90" width="64" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="274" y="109" text-anchor="middle" opacity="0.75">sam #1</text> <text fill="currentColor" x="274" y="132" text-anchor="middle">in flight</text></g> <g><rect stroke="currentColor" fill="none" x="168" y="90" width="64" height="30" rx="5"></rect><text fill="currentColor" x="200" y="109" text-anchor="middle" opacity="1">sam #2</text></g> <g><rect stroke="currentColor" fill="none" x="42" y="202" width="112" height="46" rx="8"></rect><text fill="currentColor" x="98" y="222" text-anchor="middle">Worker 1</text> <text fill="currentColor" x="98" y="238" text-anchor="middle">consumer</text></g> <g><rect stroke="currentColor" fill="none" x="164" y="202" width="112" height="46" rx="8"></rect><text fill="currentColor" x="220" y="222" text-anchor="middle">Worker 2</text> <text fill="currentColor" x="220" y="238" text-anchor="middle">consumer</text></g></svg>

Each group is a line of its own. While ana #1 is in flight, SQS hands out nothing else from acct\_ana, so her entries run in order. acct\_sam moves at the same time, on Worker 2.

FIFO queues trade throughput for that guarantee. Use one when the order of messages for the same thing matters, like the entries in one account’s ledger.

## Send many messages at once

To forward a `Stream` of messages, like a table’s changes or a file’s lines, bind `SQS.QueueSink`. It batches messages ten at a time into `sqs:SendMessageBatch` calls:

```typescript
const sink = yield* AWS.SQS.QueueSink(Jobs);
yield* Stream.fromIterable(keys).pipe(
  Stream.map((key) => JSON.stringify({ type: "thumbnail", key })),
  Stream.run(sink),
);
```

`QueueSinkHttp` is built on `SendMessageBatchHttp`, so provide both: `QueueSinkHttp` with `Layer.provideMerge(AWS.SQS.SendMessageBatchHttp)` beneath it.

When one event should reach several queues or functions, publish it to an [SNS Topic](sns.md) instead, and subscribe each Queue to the Topic.
