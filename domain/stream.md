---
url: https://alchemy.run/domain/stream
title: "Stream"
description: "How a stream keeps a keyed, append-only log that a projection writes and clients read through an Api, with its latest entry as the current value."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A stream is a keyed, append-only log: one ordered list of entries per key, kept for good and readable from any point. A [projection](projection.md) writes it; clients read it through an [Api](api.md), with the same middleware and `validate` as a view.

Its latest entry is its current value. So one stream answers both “what is it now?” and “tell me every change”, which is what a client syncing state needs: the current value first, then everything after it.

## A read model you publish entry by entry

A view’s state is shaped by its `evolve`, and its fold belongs to the model. A stream is for when you want to decide, in code, exactly what each client-visible entry contains and when one is written. The projection that writes it can read any source, combine a view’s state with an event, skip changes no client cares about, and publish a compact entry shaped for the screen.

The bank’s account card is an example. The card on a customer’s home screen shows the owner, the balance and whether the account is frozen. It changes when any of those change, and not when anything else does. A stream of `CardChanged` entries is that card, with its whole history:

<svg viewBox="0 0 700 232" role="img" aria-label="One log per account. Its last entry is the card a client shows."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 594 88 L 594 138" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">CardChanged</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">balance 100</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">CardChanged</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">balance 70</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">CardChanged</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">frozen</text></g></g> <g transform="translate(512, 144)"><g style="--hue:#e6d3a8"><path fill="none" d="M 12 0 H 158 Q 164 0 164 6 V 58 Q 164 64 158 64 H 12 L 0 32 Z"></path><g transform="translate(56.4, 21) scale(0.5833333333333334)"><path fill="none" d="M11 4a7 7 0 1 0 0 14a7 7 0 1 0 0-14z"></path><path stroke="currentColor" fill="none" d="M20 20l-4-4"></path></g><text fill="currentColor" x="76.4" y="28" dominant-baseline="middle">peek</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the card now</text></g></g></svg>

EventQuery

One log per account. Its last entry is the card a client shows.

Use a view when a fold describes the read model well. Use a stream when a projection should choose each entry, and clients should follow those entries live.

## Declare a stream

A stream names its key and the events its entries may be. The example keeps a card per account, one entry each time the card changes.

```typescript
export class CardChanged extends Domain.Event("CardChanged", {
  customerId: CustomerId,
  owner: Schema.String,
  balance: Cents,
  frozen: Schema.Boolean,
}) {}
```

```typescript
export class AccountCard extends Domain.Stream("AccountCard", {
  key: Account,
  entries: [CardChanged],
  middleware: [Session],
  rejects: [NotOwner],
```

Add it to the model’s `streams`.

A stream can also be keyed by a string Schema, like an org id, with one log per value. Its writer is then keyed by the same Schema, and reads take the value: `bank.OrgFeed.peek(orgId)`. See [Key a view by a value](view.md#key-a-view-by-a-value).

## Write it from a projection

A stream has exactly one writer: a projection that lists it in `writes`, keyed by the same aggregate. `AccountCards` reads the account summary, whose events arrive with the summary’s state after each one, and appends a card for every change.

```typescript
export const AccountCardsLive = AccountCards.toLayer(
  Effect.gen(function* () {
    const cards = yield* AccountCard;
    return Effect.fn(function* (batch) {
      for (const input of batch) {
        const summary = input.state;
        if (!summary?.customerId || !summary.owner) continue;
        yield* cards.append(
          input,
          new CardChanged({
            customerId: summary.customerId,
            owner: summary.owner,
            balance: summary.balance,
            frozen: summary.frozen,
          }),
        );
      }
    });
  }),
);
```

`append` takes the input being handled. That’s how the stream knows which key it belongs to, and how it keeps writes exactly once: each append is identified by the input’s place in the projection’s log, so a redelivered or replayed input appends nothing twice.

<svg viewBox="0 0 944 112" role="img" aria-label="Each change to the summary reaches the projection, which appends one card to the account's stream."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 56 C 713 56, 713 56, 750 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">MoneyWithdrawn</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">AccountCards</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">projection</text></g></g> <g transform="translate(756, 24)"><g style="--hue:#9aa7c7"><rect stroke="currentColor" fill="none" x="8" y="8" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M3 7h11"></path><path stroke="currentColor" fill="none" d="M3 12h17"></path><path stroke="currentColor" fill="none" d="M3 17h11"></path><path stroke="currentColor" fill="none" d="M17 9l3 3-3 3"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">AccountCard a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">stream</text></g></g></svg>

AggregateViewProjectionStream

Each change to the summary reaches the projection, which appends one card to the account's stream.

## One writer per key

Order per key only holds if each key has a single writer. The writer says so in its declaration:

```typescript
writes: [AccountCard],
```

The model checks this when it’s built: a stream listed in two projections’ `writes`, or written by a projection keyed differently, is a definition error.

## Read it

Exposed on an Api, a stream offers four reads for a key.

```typescript
export const cardReads = Effect.fn(function* (bank: CustomerClient, accountId: AccountId) {
  // The card now: the latest entry, or null.
  const now = yield* bank.AccountCard.peek(accountId);

  // Every new card, starting with the current one.
  yield* bank.AccountCard.watch(accountId).pipe(
    Stream.take(2),
    Stream.runForEach(({ state: card }) => Effect.log(\`balance ${card?.event.balance}\`)),
  );

  // History from a position, then live; and a page of it.
  yield* bank.AccountCard.events(accountId, { after: 0 }).pipe(Stream.take(1), Stream.runDrain);
  const page = yield* bank.AccountCard.list(accountId, { order: "desc", take: 10 });
  return [now, page.entries.length] as const;
});
```

Over `Domain.protocolWebSocket`, `watch` and `events` are WebSockets to the Durable Object that owns the key, which pushes each new entry as it’s appended.

`peek` also takes `atLeast`, a `stream` and `version`, and waits until the stream’s writer has handled that commit. It only covers sources the writer consumes directly. `AccountCards` reads accounts through the `AccountSummary` view, so an account command’s receipt is not one its stream can wait for.

## Decide who may read it

`validate` judges every read against the key’s latest entry, and runs again on each new entry while a client watches.

```typescript
validate: Effect.fn(function* (_request, latest) {
  yield* mustOwn(latest?.event.customerId ?? null);
}),
```

## Use it as a value

Because `peek` returns the latest entry, a stream whose entries are full snapshots, like the card, is a value with a history.

```typescript
const now = yield* bank.AccountCard.peek(accountId);
```

There’s no separate “current value” to keep in sync, and no race between two writers setting it: there’s one writer, and its entries are ordered.

## Test it

`Story.streamed` checks the entries a projection appended since the last check.

```typescript
streamed(
  AccountCard,
  a1,
  new CardChanged({ customerId: c1.id, owner: "sam", balance: 7_500, frozen: false }),
),
```

Next, [Policy](policy.md): the consumer that acts on the world rather than recording it.
