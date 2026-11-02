---
url: https://alchemy.run/domain/projection
title: "Projection"
description: "How a projection consumes events into a store you own, in order per key, batched and replayable, and why it never sends commands."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A projection consumes events into a store you own: a database table, a search index, a warehouse, or a [`Domain.Stream`](stream.md). Where a [view](view.md) is a read model the platform stores and serves for you, a projection writes wherever you choose and leaves serving it to you.

It shares the view’s routing and ordering, and differs from a [policy](policy.md) in one way that shapes everything else: a projection never sends commands. Its only effect is on its own store, so it can always be replayed.

## When a view isn’t the right store

A view answers one question about one key, and the model serves it. That covers most screens. Some questions don’t fit that shape. An accountant wants every money movement across every account this quarter, filtered by amount and sorted by date. A search box wants to find a customer by any part of their name. A data team wants the whole history in their warehouse. Each needs a store built for its kind of query, a SQL table, a search index, a warehouse, and each is a store you run.

A projection is how events reach a store like that. It’s still a read model in the CQRS sense: built from events, and never the source of truth. Add one to a model that already has history, and it receives that history first, so its store starts complete (see [Consumers added later receive the history](model.md#consumers-added-later-receive-the-history)).

<svg viewBox="0 0 700 204" role="img" aria-label="The same events feed a view the model serves and a table you query yourself."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 102 C 225 102, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 102 C 225 102, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 70)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">Account events</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the record</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">served for you</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Ledger</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">into your store</text></g></g> <g transform="translate(512, 70)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">ledger table</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">query it your way</text></g></svg>

EventViewProjection

The same events feed a view the model serves and a table you query yourself.

Reach for a view first. Reach for a projection when the query needs a store a view doesn’t give you, or when the data has to leave the model altogether.

## Declare what it consumes

A projection names its sources, the key that partitions it, and the events it consumes. In the example bank, a ledger records every money movement on an account.

```typescript
export class Ledger extends Domain.Projection("Ledger", {
  from: [Account],
  key: Account,
  on: [
    MoneyDeposited,
    MoneyWithdrawn,
    TransferDebited,
    TransferCredited,
    TransferRefunded,
    SettlementReceived,
  ],
}) {}
```

Add it to the model’s `projections`, like a policy.

<svg viewBox="0 0 700 112" role="img" aria-label="The account commits first. Shortly after, the projection writes the movement to a table outside the model."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">MoneyWithdrawn</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Ledger a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">in order</text></g></g> <g transform="translate(512, 24)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">ledger table</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">D1</text></g></svg>

AggregateProjection

The account commits first. Shortly after, the projection writes the movement to a table outside the model.

## Implement it as a Layer

The handler lives in a Layer, built once with everything it needs. It always receives a batch: an array of inputs for one key, in order.

```typescript
export const LedgerLive = Ledger.toLayer(
  Effect.gen(function* () {
    const store = yield* LedgerStore;
    return Effect.fn(function* (batch) {
      yield* store.record(
        batch.map(({ key, event, envelope }) => ({
          id: \`${envelope.stream}:${envelope.seq}\`,
          accountId: key.id,
          kind: event._tag,
          amount: event.amount,
          balanceAfter: event.balanceAfter,
          at: DateTime.toEpochMillis(envelope.at),
        })),
      );
    });
  }),
  { batch: { size: 100, wait: "1 second" } },
);
```

Each input carries the `key` it was routed to, the `event`, its `envelope`, the `source` it came from, and, for a view source, the view’s `state` after the event.

## Depend on a store, not a database

The projection asks for `LedgerStore`, a service you define in the domain package, rather than a D1 binding. The server package provides it with D1; a story provides it with memory.

```typescript
export class LedgerStore extends Context.Service<
  LedgerStore,
  {
    readonly record: (rows: ReadonlyArray<LedgerRow>) => Effect.Effect<void, never, RuntimeContext>;
    readonly forAccount: (
      accountId: string,
    ) => Effect.Effect<ReadonlyArray<LedgerRow>, never, RuntimeContext>;
  }
>()("Bank/LedgerStore") {}
```

Its methods require `RuntimeContext`, because a database binding only exists inside a running Worker. A projection’s handler may require it; the platform runs handlers inside the runtime.

## Make every write idempotent

Inputs arrive at least once. If a handler fails halfway, or its host restarts, the same inputs run again, so a write must be safe to repeat. The ledger keys each row by the event it came from, `stream:seq`, and ignores a row it already has.

```typescript
"INSERT INTO ledger (id, account_id, kind, amount, balance_after, at) VALUES (?, ?, ?, ?, ?, ?) ON CONFLICT (id) DO NOTHING",
```

The platform helps on its side too: every key keeps an input log, and a delivery it has already accepted is dropped on arrival.

## Partition it by key

`key` decides which inputs are handled in order. Inputs for one key are handled one batch at a time, in the order they were accepted; different keys run in parallel.

<svg viewBox="0 0 456 204" role="img" aria-label="One ordered partition per key. Keys don't wait for each other."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">events</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">events</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Ledger a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one at a time</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Ledger a-2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one at a time</text></g></g></svg>

AggregateProjection

One ordered partition per key. Keys don't wait for each other.

A source keyed by the same aggregate routes itself. For any other source, `keyOf` names the key, by event tag or source name, and can send one event to several keys with named routes, exactly as a [view’s `keyOf`](view.md#route-one-event-to-several-keys) does. Each input then carries its `route`.

The key can also be a string Schema, such as an org id, when there is no aggregate to partition by. `keyOf` then returns the value, and the handler gets it as `key`.

```typescript
export class OrgLedger extends Domain.Projection("OrgLedger", {
  from: [Account],
  key: OrgId,
  keyOf: { Account: ({ event }) => event.orgId },
  on: [MoneyDeposited],
}) {}
```

Within a key, events from one source stay in that source’s order. Events from different sources have no order between them: the model is distributed on purpose, and there is no total order to give.

## Batch it on its Layer

Batching changes how a projection runs, not what it means, so it lives on the Layer. `batch: { size, wait }` hands the handler up to `size` inputs of one key at once, waiting up to `wait` for a batch to fill. Without it, each input is handled on its own, as soon as it arrives.

```typescript
{ batch: { size: 100, wait: "1 second" } },
```

The handler’s code is the same either way: it always receives an array. Changing the batch size is pure tuning.

`version` is tuning too. Bump it and the projection runs again over the whole history on the next deploy, for example after its table was lost, or once the handler writes a new shape. Empty or migrate the table first if a replayed write wouldn’t overwrite the old row.

```typescript
{ batch: { size: 100, wait: "1 second" }, version: 1 },
```

## It never sends commands

A projection’s effects stay in its own store. That’s what makes it safe to replay: run it again over the same inputs, and the only thing that happens is the same rows are written again.

A policy is the opposite. It acts on the world, through commands and external systems, so it handles each new event once and never replays. When you need both, split the work: the projection writes the store, and a policy on the same events sends the commands.

<svg viewBox="0 0 700 204" role="img" aria-label="The same event, consumed twice: once into a store that can be rebuilt, once into an action that can't be undone."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 102 C 225 102, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 102 C 225 102, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 148, 506 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 70)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">MoneyWithdrawn</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Account a-1</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Ledger</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">replayable</text></g></g> <g transform="translate(512, 24)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">ledger table</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">your store</text></g> <g transform="translate(268, 116)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">FraudReview</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">never replays</text></g></g> <g transform="translate(512, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">FreezeAccount</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">acts on the world</text></g></g></svg>

CommandEventProjectionPolicy

The same event, consumed twice: once into a store that can be rebuilt, once into an action that can't be undone.

## Read your own write

Your store is eventually consistent with the model. A caller who has just sent a command and wants to read the result from your store waits for the projection first.

```typescript
export const ledgerAfterWithdrawal = Effect.fn(function* (accountId: AccountId) {
  const accounts = yield* Account;
  const receipt = yield* accounts.send(accountId, new WithdrawMoney({ amount: 2_000 }));
  yield* Ledger.caughtUp(receipt);
  return yield* (yield* LedgerStore).forAccount(accountId);
});
```

`caughtUp` waits until the projection has handled every event in the receipt that it consumes directly from that aggregate.

## Serve it yourself

A projection’s store is yours, so its API is too. The example bank serves the ledger from a plain route on its Worker, authorizing the caller itself.

```typescript
const summary = yield* summaries.query(Account.ref(accountId)).pipe(Effect.orDie);
const owner = Option.getOrNull(summary)?.customerId;
if (actor.value._tag === "Customer" && owner !== actor.value.customerId) {
  return HttpServerResponse.empty({ status: 404 });
}
return yield* HttpServerResponse.json(yield* ledger.forAccount(accountId));
```

Clients that want to know when to refetch can follow the same events the projection consumed, through the Api.

To have the model serve a projection’s output for you, with live updates and `validate`, write a [stream](stream.md) instead.

## Test it

Projections run for real in a [story](story.md). Give the story a store you can read back.

```typescript
const ledger = memoryLedger();
yield* ledgerStory(
  given(a1, opened(), new MoneyDeposited({ amount: 10_000, balanceAfter: 10_000 })),
  as(sam),
  when(a1, new WithdrawMoney({ amount: 2_500 })),
  then(a1, new MoneyWithdrawn({ amount: 2_500, balanceAfter: 7_500 })),
).pipe(Effect.provide(ledger.layer));
```

Next, [Stream](stream.md): a store the model keeps and serves for a projection.
