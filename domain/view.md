---
url: https://alchemy.run/domain/view
title: "View"
description: "How a view folds events into a read model per key, emits events of its own, combines other views, and serves reads."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A view answers a question: what’s the balance of this account, what’s on its statement, what does this customer hold across all their accounts. It reads events, folds them into state, and keeps that state current as new events arrive.

Aggregates hold only what their rules need. Views hold everything people want to see, each shaped for one question. This chapter starts with why the two are kept apart and how to design a view from a screen, then builds views up from the simplest case.

## Deciding and answering need different shapes

An account needs exactly three things to decide a withdrawal: whether it’s open, who owns it, and its balance. A customer’s app needs much more. It shows a statement, line by line. It shows a dashboard with every account the customer holds. A support agent sees an account’s freezes, closures and owner changes, and a fraud check needs today’s total withdrawals.

Putting all of that in the `Account` aggregate would make every rule harder to read, and every new screen a change to the code that guards the money. So the model keeps them apart. Aggregates are the **write side**, holding just what their rules need. Views are the **read side**: as many as there are questions, each built from the same events and shaped for the question it answers.

<svg viewBox="0 0 700 296" role="img" aria-label="One aggregate, many views. Adding a view never touches a rule."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 148, 506 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 240, 506 240" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 116)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">open, owner, balance</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Events</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the shared record</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the balance now</text></g></g> <g transform="translate(512, 116)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(36.9, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="56.9" y="28" dominant-baseline="middle">Statement</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one line per movement</text></g></g> <g transform="translate(512, 208)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">DailyOutflow</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">today's withdrawals</text></g></g></svg>

AggregateEventView

One aggregate, many views. Adding a view never touches a rule.

## Design a view from its question

Start from the screen, not from the events. Write down the question it answers and the data it shows: “what’s on this account’s statement?” needs a list of lines, each with a kind, an amount and the balance after. Then decide what one instance of the view is about. A statement is about one account, so it’s keyed by account. A dashboard is about one customer, so it’s keyed by customer.

Last, work backwards to the events that change that data. A statement line comes from each deposit and each withdrawal. If the data a screen needs isn’t in any event, that’s worth noticing: either the event is missing a fact the business cares about, or the screen is asking for something the business never recorded.

## Fold one aggregate’s events

The simplest view follows one aggregate. `AccountSummary` reads the account’s events and keeps its owner and balance.

```typescript
export class AccountSummary extends Domain.View("AccountSummary", {
  from: [Account],
  key: Account,
  state: Schema.Struct({ owner: Schema.NullOr(Schema.String), balance: Schema.Int }),
  initial: { owner: null, balance: 0 },
  evolve: {
    AccountOpened: (v, e) => ({ ...v, owner: e.owner }),
    MoneyDeposited: (v, e) => ({ ...v, balance: e.balanceAfter }),
    MoneyWithdrawn: (v, e) => ({ ...v, balance: e.balanceAfter }),
  },
}) {}
```

`from` lists where its events come from. `key` says what one instance of the view is about: here, one account. So every account gets its own summary, and an event from account `a-1` updates only the summary for `a-1`.

<svg viewBox="0 0 456 204" role="img" aria-label="One summary per account, each fed by that account's events."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">aggregate</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">aggregate</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Summary a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">AccountSummary</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Summary a-2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">AccountSummary</text></g></g></svg>

AggregateView

One summary per account, each fed by that account's events.

`evolve` works like an aggregate’s: one handler per event, returning the next state. A view only handles the events it names. The rest pass it by, so adding an event to the account never breaks a view that doesn’t care about it.

## Views catch up after the commit

A view is never updated in the same step as the aggregate. The account commits its events first, then they’re delivered to the view. Each view receives every event it reads at least once, in the order each source recorded them, and drops any it has already applied.

So a view is always correct, but it can be a moment behind. Most screens never notice. One that has just sent a command and wants to show its result waits for the view to catch up, using the command’s receipt.

```typescript
export const depositThenRead = Effect.fn(function* (bank: CustomerClient, accountId: AccountId) {
  const receipt = yield* bank.DepositMoney(accountId, { amount: 500 });
  const summary = yield* bank.AccountSummary.get(accountId, {
    atLeast: { stream: receipt.stream, version: receipt.version },
  });
  return summary?.balance;
});
```

`atLeast` makes `get` wait until the summary has applied that version of the account’s stream. If it hasn’t within 10 seconds, the read fails with `ConsistencyTimeout`.

A view added to a model that already has events receives them first, so a new screen starts from the whole history (see [Consumers added later receive the history](model.md#consumers-added-later-receive-the-history)).

## Emit events of your own

A view can record events as it folds, which is how you build lists like a statement. Instead of `evolve`, `Statement` has `emit`: for each deposit or withdrawal, it records one `StatementLine`.

```typescript
export class Statement extends Domain.View("Statement", {
  from: [Account],
  key: Account,
  events: [StatementLine],
  emit: {
    MoneyDeposited: (e) =>
      new StatementLine({ kind: "deposit", amount: e.amount, balance: e.balanceAfter }),
    MoneyWithdrawn: (e) =>
      new StatementLine({ kind: "withdrawal", amount: e.amount, balance: e.balanceAfter }),
  },
}) {}
```

`StatementLine` is shaped for the reader, not the account. A client lists the lines a page at a time, or follows them live.

A view that declares no `events` emits every event it applies, each paired with its state after that event. That makes any view a source for another.

## Build on other views

A customer’s dashboard shows every account they own, with each balance. No single account knows that, and the customer aggregate doesn’t track balances. But `AccountSummary` already knows each account’s balance and owner, so the dashboard reads from it.

<svg viewBox="0 0 456 296" role="img" aria-label="The dashboard is keyed by customer and fed by every summary that customer owns."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 240 C 225 240, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Summary a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">AccountSummary</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Summary a-2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">AccountSummary</text></g></g> <g transform="translate(24, 208)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">Customer c-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">aggregate</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">Dashboard c-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">CustomerDashboard</text></g></g></svg>

AggregateView

The dashboard is keyed by customer and fed by every summary that customer owns.

The dashboard is keyed by customer, but a summary’s events come from accounts. `keyOf` tells the view which customer each one belongs to, using the summary’s state after the event.

```typescript
keyOf: {
  AccountSummary: ({ state }) => (state?.customerId ? Customer.ref(state.customerId) : undefined),
```

Returning `undefined` means the event belongs to no dashboard, like a summary for an account that hasn’t been opened yet. The handler for those events receives the same state, and records that account’s balance.

```typescript
AccountSummary: (d, _, { source, state }) => ({
  ...d,
  balances:
    state && !state.closed
      ? { ...d.balances, [source.id]: state.balance }
      : without(d.balances, source.id),
}),
```

## Key a view by a value

Some read models are about something the model has no aggregate for, like an organization that owns many accounts. Key the view by a string Schema instead of an aggregate. Each value gets its own instance, and `keyOf` returns the value itself, not a ref.

```typescript
export const OrgId = Schema.String.pipe(Schema.brand("OrgId"));

export class OrgSchedule extends Domain.View("OrgSchedule", {
  from: [Tour],
  key: OrgId,
  keyOf: { Tour: ({ event }) => event.orgId },
  state: Schema.Struct({ tours: Schema.Record(Schema.String, Schema.String) }),
  initial: { tours: {} },
  evolve: {
    TourBooked: (s, e, { source }) => ({ tours: { ...s.tours, [source.id]: e.unit } }),
  },
}) {}
```

Reads take the value too: `schedules.watch(orgId)`, `client.OrgSchedule.get(orgId)`, and `Story.view(OrgSchedule, orgId, ...)`. In handlers, `key` is the value, and a view, policy or projection that consumes this view gets it as `source`. Policies, projections and streams take a Schema `key` the same way. A view keyed by the same Schema follows this one without `keyOf`, so use a branded Schema to keep different kinds of ids apart.

## Route one event to several keys

Some events matter to more than one key. A transfer request names two accounts, and both of their transfer histories should show it, one as money out and one as money in.

`keyOf` can return named routes. Each route picks a key, and each has its own handler.

```typescript
keyOf: {
  TransferRequested: {
    out: ({ event }) => Account.ref(event.from),
    in: ({ event }) => Account.ref(event.to),
  },
},
events: [TransferMoved],
emit: {
  TransferRequested: { out: moved("out"), in: moved("in") },
},
```

One `TransferRequested` updates both accounts: the sender’s as `out` and the recipient’s as `in`.

## Read it

Through an [Api](api.md), a view offers these reads for one key. `get` returns its current state, or `null` if nothing has happened to that key yet. `list` returns a page of its emitted events. `watch` streams the state now and after every change, each with its cursor. `events` streams its emitted events: history first, then a `CaughtUp` frame, then new ones as they’re recorded. `snapshot` returns the state with the cursor it is valid at.

```typescript
export const statementPage = Effect.fn(function* (bank: CustomerClient, accountId: AccountId) {
  const page = yield* bank.Statement.list(accountId, { order: "desc", take: 10 });
  return page.entries.map(({ event }) => \`${event.kind} ${event.amount}\`);
});
```

A `where` filter narrows any read, like only deposits over a threshold. A filter can only narrow what you’re already allowed to read; it never grants access.

## Know when a live read is current

`events` sends the key’s history, then a `CaughtUp` frame, then new entries as they’re recorded. Once `CaughtUp` arrives, the screen shows the key’s current state and can stop showing “loading”.

```typescript
yield* bank.Statement.events(accountId).pipe(
  Stream.runForEach((frame) =>
    Domain.Read.isCaughtUp(frame) ? markLive(frame.cursor) : addLine(frame),
  ),
);
```

`Stream.filter(Domain.Read.isEvent)` drops the frame when you don’t need it.

`watch` sends each state with its cursor, `{ state, cursor }`. Pass a cursor as `after` to `events` and the feed starts right after it: nothing arrives twice, and nothing is missed. That’s how a screen that rendered a watched state follows the events from there, and how a dropped feed resumes from its last `CaughtUp`. Over `protocolWebSocket`, a read that reconnects sends `CaughtUp` again after the entries it missed.

### Render a snapshot, then follow it

`snapshot` returns a key’s state together with its `cursor`: the `seq` of the last event the key had emitted when that state was read. Render the state, then follow `events` from the cursor. Every change after the snapshot arrives once, and none is missed, even when it was written between the two reads:

```typescript
export const followSummary = Effect.fn(function* (
  bank: CustomerClient,
  accountId: AccountId,
  render: (balance: number | undefined) => Effect.Effect<void>,
) {
  const { state, cursor } = yield* bank.AccountSummary.snapshot(accountId);
  yield* render(state?.balance);
  yield* bank.AccountSummary.events(accountId, { after: cursor }).pipe(
    Stream.filter(Domain.Read.isEvent),
    Stream.runForEach((entry) => render(entry.state?.balance)),
  );
});
```

A server can read the snapshot while it renders a page and send it with the page, so the browser shows data at once and starts its feed from the cursor. `snapshot` and `list` take `atLeast`, like `get`: pass a command’s `stream` and `version` to wait until the view has applied it.

## Decide who may read it

A view that’s exposed through an Api declares `middleware` to identify the caller, and `validate` to judge each read against the view’s state.

```typescript
validate: Effect.fn(function* (_request, summary) {
  yield* mustOwn(summary?.customerId ?? null);
}),
```

`validate` runs on every read, and again on every change while a client watches. If an account changes owner, the previous owner’s watch ends with `NotOwner`. See [Views as security boundaries](view-security.md).

Every key keeps an input log: each delivery it applied, in order, kept for good. A view’s state is a fold of that log.

Next, [Projection](projection.md): a read model in a store you own, when a view’s built-in store isn’t the right one.
