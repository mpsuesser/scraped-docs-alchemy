---
url: https://alchemy.run/aws/messaging/dynamodb-streams
title: "DynamoDB Streams"
description: "How a DynamoDB Stream records every change to a table in order, what each change record carries, and how a Function consumes changes and checkpoints at the first failure."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A DynamoDB Stream is a table’s own change log. Every insert, update and delete becomes a **change record**, kept for 24 hours, in the order the changes happened to each item. A Function reading the stream sees every change to the table, including ones made by code that never told anyone.

## Every write becomes a change record

The table writes the record as part of the change, so there’s no second write to forget or to fail. Records for one item stay in order. Records for different items may be read in parallel.

<svg viewBox="0 0 647 94" style="max-width: 744px" role="img" aria-label="Every write to the table appends a change record, as part of the write. Records for one item stay in order, and the consumer reads them shortly after."><defs><marker id="alc-log-arrow-v2z1l8" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><text fill="currentColor" x="4" y="52">Orders</text> <text fill="currentColor" x="4" y="66">table stream</text> <g><rect stroke="currentColor" fill="none" x="98.80000000000001" y="38" width="67.6" height="30" rx="5"></rect><text fill="currentColor" x="132.60000000000002" y="57" text-anchor="middle">INSERT #1</text> <text fill="currentColor" x="132.60000000000002" y="81" text-anchor="middle" opacity="0.7">0</text></g> <g><rect stroke="currentColor" fill="none" x="170.4" y="38" width="67.6" height="30" rx="5"></rect><text fill="currentColor" x="204.2" y="57" text-anchor="middle">MODIFY #1</text> <text fill="currentColor" x="204.2" y="81" text-anchor="middle" opacity="0.7">1</text></g> <g><rect stroke="currentColor" fill="none" x="242" y="38" width="67.6" height="30" rx="5"></rect><text fill="currentColor" x="275.8" y="57" text-anchor="middle">INSERT #2</text> <text fill="currentColor" x="275.8" y="81" text-anchor="middle" opacity="0.7">2</text></g> <g><rect stroke="currentColor" fill="none" x="313.6" y="38" width="67.6" height="30" rx="5"></rect><text fill="currentColor" x="347.40000000000003" y="57" text-anchor="middle">MODIFY #2</text> <text fill="currentColor" x="347.40000000000003" y="81" text-anchor="middle" opacity="0.7">3</text></g> <g><rect stroke="currentColor" fill="none" x="385.2" y="38" width="67.6" height="30" rx="5"></rect><text fill="currentColor" x="419" y="57" text-anchor="middle">REMOVE #1</text> <text fill="currentColor" x="419" y="81" text-anchor="middle" opacity="0.7">4</text></g> <rect stroke="currentColor" fill="none" x="456.8" y="38" width="67.6" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="633" y1="53" x2="528.4" y2="53" marker-end="url(#alc-log-arrow-v2z1l8)"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="347.40000000000003" y1="22" x2="347.40000000000003" y2="34" marker-end="url(#alc-log-arrow-v2z1l8)"></line><text fill="currentColor" x="347.40000000000003" y="18" text-anchor="middle">Indexer</text></g></g> <text fill="currentColor" x="639" y="30" text-anchor="end">every write</text></svg> <svg viewBox="0 0 179.79999999999998 280" style="max-width: 207px" role="img" aria-label="Every write to the table appends a change record, as part of the write. Records for one item stay in order, and the consumer reads them shortly after."><g><text fill="currentColor" x="24" y="12">Orders</text> <text fill="currentColor" x="24" y="26">table stream</text> <g><text fill="currentColor" x="18" y="53" text-anchor="end" opacity="0.7">0</text> <rect stroke="currentColor" fill="none" x="24" y="34" width="71.6" height="30" rx="5"></rect><text fill="currentColor" x="59.8" y="53" text-anchor="middle">INSERT #1</text></g> <g><text fill="currentColor" x="18" y="89" text-anchor="end" opacity="0.7">1</text> <rect stroke="currentColor" fill="none" x="24" y="70" width="71.6" height="30" rx="5"></rect><text fill="currentColor" x="59.8" y="89" text-anchor="middle">MODIFY #1</text></g> <g><text fill="currentColor" x="18" y="125" text-anchor="end" opacity="0.7">2</text> <rect stroke="currentColor" fill="none" x="24" y="106" width="71.6" height="30" rx="5"></rect><text fill="currentColor" x="59.8" y="125" text-anchor="middle">INSERT #2</text></g> <g><text fill="currentColor" x="18" y="161" text-anchor="end" opacity="0.7">3</text> <rect stroke="currentColor" fill="none" x="24" y="142" width="71.6" height="30" rx="5"></rect><text fill="currentColor" x="59.8" y="161" text-anchor="middle">MODIFY #2</text></g> <g><text fill="currentColor" x="18" y="197" text-anchor="end" opacity="0.7">4</text> <rect stroke="currentColor" fill="none" x="24" y="178" width="71.6" height="30" rx="5"></rect><text fill="currentColor" x="59.8" y="197" text-anchor="middle">REMOVE #1</text></g> <rect stroke="currentColor" fill="none" x="24" y="214" width="71.6" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="59.8" y1="264" x2="59.8" y2="246" marker-end="url(#alc-log-arrow-n-v2z1l8)"></line><text fill="currentColor" x="67.8" y="274">every write</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="115.6" y1="157" x2="99.6" y2="157" marker-end="url(#alc-log-arrow-n-v2z1l8)"></line><text fill="currentColor" x="119.6" y="161">Indexer</text></g></g><defs><marker id="alc-log-arrow-n-v2z1l8" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs></svg>

Every write to the table appends a change record, as part of the write. Records for one item stay in order, and the consumer reads them shortly after.

Each record says what happened, `INSERT`, `MODIFY` or `REMOVE`, the item’s keys, and, depending on the stream’s **view type**, the item as it was before, after, or both.

Subscribe a Function from Construction. Each invocation receives a batch of change records:

```typescript
export default class Indexer extends AWS.Lambda.Function<Indexer>()(
  "Indexer",
  { main: import.meta.url, functionUrl: false },
  Effect.gen(function* () {
    const orders = yield* Orders;
    yield* AWS.DynamoDB.consumeTableChanges(orders, (event) =>
      Effect.forEach(
        event.Records,
        (record) => Console.log(\`${record.eventName}: ${record.dynamodb.Keys?.pk?.S}\`),
        { discard: true },
      ),
    );
    return {};
  }).pipe(Effect.provide(AWS.Lambda.TableEventSource)),
) {}
```

There’s nothing to turn on first. Deploy enables the stream on the table, creates the event source mapping, and grants the Function read access to the stream.

## Choose what each record carries

By default a record carries the item’s keys. Ask for the item itself with `streamViewType`:

```typescript
yield* AWS.DynamoDB.consumeTableChanges(orders, { streamViewType: "NEW_AND_OLD_IMAGES" }, handle);
```

`NEW_IMAGE` carries the item after the change, `OLD_IMAGE` before it, and `NEW_AND_OLD_IMAGES` both, which lets a consumer see exactly which attributes changed. Pick the smallest one your consumer needs.

The images are in DynamoDB’s attribute format. Type the event with your item’s shape to read them:

```typescript
(event: AWS.DynamoDB.StreamEvent<{ pk: { S: string }; total: { N: string } }>) =>
  Effect.forEach(
    event.Records,
    (record) => Console.log(record.dynamodb.NewImage?.total.N ?? "(deleted)"),
    { discard: true },
  )
```

## Start from the right place

A new subscription starts with changes made after it exists, `startingPosition: "LATEST"`. To process what the stream still holds from the last 24 hours, start from `"TRIM_HORIZON"`:

```typescript
yield* AWS.DynamoDB.consumeTableChanges(
  orders,
  { streamViewType: "NEW_IMAGE", startingPosition: "TRIM_HORIZON", batchSize: 10 },
  handle,
);
```

`batchSize` caps the records per invocation. Larger batches cost fewer invocations. Smaller ones retry less work when one fails.

## Checkpoint at the first failure

Lambda moves its place in the stream forward after each batch succeeds. When a batch fails, the batch is retried, and changes behind it wait, so they’re never applied out of order.

Report the first record that failed instead, and Lambda keeps everything before it:

```typescript
(event) =>
  Effect.gen(function* () {
    for (const record of event.Records) {
      const ok = yield* writeToIndex(record).pipe(
        Effect.as(true),
        Effect.catchCause(() => Effect.succeed(false)),
      );
      if (!ok) return { batchItemFailures: [{ itemIdentifier: record.dynamodb.SequenceNumber! }] };
    }
  })
```

A record that can never succeed blocks the stream for its items until it expires. Catch errors that retrying won’t fix, and move on.

## What streams are for

Use a stream to keep something else in step with the table: a search index, a cache, a copy in a warehouse, a running total. Use it to react to changes, like sending an email when an order’s status becomes `shipped`, without the code that changed it having to know.

To hand changes to other consumers, forward them to a [Queue](sqs.md#send-many-messages-at-once) or a [Topic](sns.md). For an ordered stream your code writes to directly, use [Kinesis](kinesis.md).
