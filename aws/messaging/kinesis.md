---
url: https://alchemy.run/aws/messaging/kinesis
title: "Kinesis"
description: "How Kinesis Data Streams keep records in order per partition key across shards, how runtimes write records and consume them in batches, and how retention, replay and checkpoints work."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

Kinesis Data Streams is an ordered log of **records**. Producers append records, and they stay in the stream for a retention period, 24 hours by default, whether or not anyone has read them. Consumers read the log in order, each keeping its own place, so several consumers can read the same records independently and any of them can go back and read again.

## Shards keep records in order

A stream is split into **shards**. Each record carries a **partition key**, and Kinesis sends every record with the same key to the same shard. Within a shard, records are kept in the order they arrived.

<svg viewBox="0 0 569.2 180" style="max-width: 655px" role="img" aria-label="The partition key picks the shard, so every record for order #1 is in shard 1, in order. A consumer reads each shard in parallel, at its own position."><defs><marker id="alc-log-arrow-n9bgip" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><text fill="currentColor" x="4" y="57">shard 1</text> <g><rect stroke="currentColor" fill="none" x="73.80000000000001" y="38" width="74" height="30" rx="5"></rect><text fill="currentColor" x="110.80000000000001" y="57" text-anchor="middle">#1 placed</text> <text fill="currentColor" x="110.80000000000001" y="81" text-anchor="middle" opacity="0.7">0</text></g> <g><rect stroke="currentColor" fill="none" x="151.8" y="38" width="74" height="30" rx="5"></rect><text fill="currentColor" x="188.8" y="57" text-anchor="middle">#3 placed</text> <text fill="currentColor" x="188.8" y="81" text-anchor="middle" opacity="0.7">1</text></g> <g><rect stroke="currentColor" fill="none" x="229.8" y="38" width="74" height="30" rx="5"></rect><text fill="currentColor" x="266.8" y="57" text-anchor="middle">#1 paid</text> <text fill="currentColor" x="266.8" y="81" text-anchor="middle" opacity="0.7">2</text></g> <g><rect stroke="currentColor" fill="none" x="307.8" y="38" width="74" height="30" rx="5"></rect><text fill="currentColor" x="344.8" y="57" text-anchor="middle">#1 shipped</text> <text fill="currentColor" x="344.8" y="81" text-anchor="middle" opacity="0.7">3</text></g> <rect stroke="currentColor" fill="none" x="385.8" y="38" width="74" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="555.2" y1="53" x2="463.8" y2="53" marker-end="url(#alc-log-arrow-n9bgip)"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="266.8" y1="22" x2="266.8" y2="34" marker-end="url(#alc-log-arrow-n9bgip)"></line><text fill="currentColor" x="266.8" y="18" text-anchor="middle">Audit</text></g></g> <g><text fill="currentColor" x="4" y="143">shard 2</text> <g><rect stroke="currentColor" fill="none" x="73.80000000000001" y="124" width="74" height="30" rx="5"></rect><text fill="currentColor" x="110.80000000000001" y="143" text-anchor="middle">#2 placed</text> <text fill="currentColor" x="110.80000000000001" y="167" text-anchor="middle" opacity="0.7">0</text></g> <g><rect stroke="currentColor" fill="none" x="151.8" y="124" width="74" height="30" rx="5"></rect><text fill="currentColor" x="188.8" y="143" text-anchor="middle">#2 paid</text> <text fill="currentColor" x="188.8" y="167" text-anchor="middle" opacity="0.7">1</text></g> <g><rect stroke="currentColor" fill="none" x="229.8" y="124" width="74" height="30" rx="5"></rect><text fill="currentColor" x="266.8" y="143" text-anchor="middle">#4 placed</text> <text fill="currentColor" x="266.8" y="167" text-anchor="middle" opacity="0.7">2</text></g> <rect stroke="currentColor" fill="none" x="307.8" y="124" width="74" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="555.2" y1="139" x2="385.8" y2="139" marker-end="url(#alc-log-arrow-n9bgip)"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="188.8" y1="108" x2="188.8" y2="120" marker-end="url(#alc-log-arrow-n9bgip)"></line><text fill="currentColor" x="188.8" y="104" text-anchor="middle">Audit</text></g></g> <text fill="currentColor" x="561.2" y="30" text-anchor="end">PutRecord</text></svg> <svg viewBox="0 0 346 232" style="max-width: 398px" role="img" aria-label="The partition key picks the shard, so every record for order #1 is in shard 1, in order. A consumer reads each shard in parallel, at its own position."><g><text fill="currentColor" x="24" y="12">shard 1</text> <g><text fill="currentColor" x="18" y="41" text-anchor="end" opacity="0.7">0</text> <rect stroke="currentColor" fill="none" x="24" y="22" width="78" height="30" rx="5"></rect><text fill="currentColor" x="63" y="41" text-anchor="middle">#1 placed</text></g> <g><text fill="currentColor" x="18" y="77" text-anchor="end" opacity="0.7">1</text> <rect stroke="currentColor" fill="none" x="24" y="58" width="78" height="30" rx="5"></rect><text fill="currentColor" x="63" y="77" text-anchor="middle">#3 placed</text></g> <g><text fill="currentColor" x="18" y="113" text-anchor="end" opacity="0.7">2</text> <rect stroke="currentColor" fill="none" x="24" y="94" width="78" height="30" rx="5"></rect><text fill="currentColor" x="63" y="113" text-anchor="middle">#1 paid</text></g> <g><text fill="currentColor" x="18" y="149" text-anchor="end" opacity="0.7">3</text> <rect stroke="currentColor" fill="none" x="24" y="130" width="78" height="30" rx="5"></rect><text fill="currentColor" x="63" y="149" text-anchor="middle">#1 shipped</text></g> <rect stroke="currentColor" fill="none" x="24" y="166" width="78" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="63" y1="216" x2="63" y2="198" marker-end="url(#alc-log-arrow-n-n9bgip)"></line><text fill="currentColor" x="71" y="226">PutRecord</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="122" y1="109" x2="106" y2="109" marker-end="url(#alc-log-arrow-n-n9bgip)"></line><text fill="currentColor" x="126" y="113">Audit</text></g></g> <g><text fill="currentColor" x="197" y="12">shard 2</text> <g><text fill="currentColor" x="191" y="41" text-anchor="end" opacity="0.7">0</text> <rect stroke="currentColor" fill="none" x="197" y="22" width="78" height="30" rx="5"></rect><text fill="currentColor" x="236" y="41" text-anchor="middle">#2 placed</text></g> <g><text fill="currentColor" x="191" y="77" text-anchor="end" opacity="0.7">1</text> <rect stroke="currentColor" fill="none" x="197" y="58" width="78" height="30" rx="5"></rect><text fill="currentColor" x="236" y="77" text-anchor="middle">#2 paid</text></g> <g><text fill="currentColor" x="191" y="113" text-anchor="end" opacity="0.7">2</text> <rect stroke="currentColor" fill="none" x="197" y="94" width="78" height="30" rx="5"></rect><text fill="currentColor" x="236" y="113" text-anchor="middle">#4 placed</text></g> <rect stroke="currentColor" fill="none" x="197" y="130" width="78" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="236" y1="216" x2="236" y2="162" marker-end="url(#alc-log-arrow-n-n9bgip)"></line><text fill="currentColor" x="244" y="226">PutRecord</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="295" y1="73" x2="279" y2="73" marker-end="url(#alc-log-arrow-n-n9bgip)"></line><text fill="currentColor" x="299" y="77">Audit</text></g></g><defs><marker id="alc-log-arrow-n-n9bgip" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs></svg>

The partition key picks the shard, so every record for order #1 is in shard 1, in order. A consumer reads each shard in parallel, at its own position.

So the guarantee is per key: every event for `order#1` arrives in the order it happened. Events for different orders may interleave. Choose the partition key as the thing whose history must stay in order, usually an entity’s id.

## Declare a stream

```typescript
import * as AWS from "@alchemy.run/aws";

export const Events = AWS.Kinesis.Stream("Events", { streamMode: "ON_DEMAND" });
```

On-demand streams add and remove shards with load, and you pay for the data you write and read. With `streamMode: "PROVISIONED"` you set `shardCount` yourself, and pay per shard per hour, which is cheaper at steady, known throughput.

Keep records longer than a day with `retentionPeriod`, up to a year, so a new consumer or a fixed bug can replay further back.

## Write records

Bind `Kinesis.PutRecord` in the producer. Its role gets `kinesis:PutRecord` on this stream:

```typescript
const putRecord = yield* AWS.Kinesis.PutRecord(Events);
yield* putRecord({
  PartitionKey: orderId,
  Data: new TextEncoder().encode(JSON.stringify({ type: "order.paid", orderId })),
});
```

`Data` is bytes, so JSON, Avro or protobuf all work. Provide `AWS.Kinesis.PutRecordHttp` to the runtime. To write up to 500 records in one call, bind `Kinesis.PutRecords`. To write a whole `Stream`, bind `Kinesis.StreamSink`.

## Consume records

A Lambda Function consumes a stream by subscribing from Construction. Lambda reads each shard in order and invokes the Function with batches:

```typescript
export default class Audit extends AWS.Lambda.Function<Audit>()(
  "Audit",
  { main: import.meta.url, functionUrl: false },
  Effect.gen(function* () {
    const events = yield* Events;
    yield* AWS.Kinesis.consumeStreamRecords(
      events,
      { startingPosition: "LATEST", batchSize: 100 },
      (event) =>
        Effect.forEach(
          event.Records,
          (record) =>
            Console.log(new TextDecoder().decode(Buffer.from(record.kinesis.data, "base64"))),
          { discard: true },
        ),
    );
    return {};
  }).pipe(Effect.provide(AWS.Lambda.StreamEventSource)),
) {}
```

Each record’s data arrives base64-encoded in the event, so decode it first. Deploy creates the event source mapping and grants the Function read access to the stream.

`startingPosition: "LATEST"` starts with records written after the subscription exists. `"TRIM_HORIZON"` starts with the oldest record still retained, replaying everything the stream holds.

## Checkpoint at the first failure

Lambda moves a shard’s **checkpoint** forward after each batch succeeds. If a batch fails, the whole batch is retried, and the shard waits. Nothing behind it is processed until it succeeds, because skipping ahead would break the order.

Report the first record that failed instead, and Lambda checkpoints everything before it and retries from there:

```typescript
(event) =>
  Effect.gen(function* () {
    for (const record of event.Records) {
      const ok = yield* handleRecord(record).pipe(
        Effect.as(true),
        Effect.catchCause(() => Effect.succeed(false)),
      );
      if (!ok) return { batchItemFailures: [{ itemIdentifier: record.kinesis.sequenceNumber }] };
    }
  }),
```

A record that can never succeed still blocks its shard until it expires. Catch errors that retrying won’t fix, record them somewhere you’ll see, and move on.

## Stream or queue

Both carry work from producers to consumers. Use a stream when order matters per key, when several consumers read the same records, or when you want to replay history. Use an [SQS Queue](sqs.md) when each message is independent work for one consumer: a failed message there is retried on its own, without holding up the rest.

A table’s own change log is a stream too. See [DynamoDB Streams](dynamodb-streams.md).
