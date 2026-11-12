---
url: https://alchemy.run/domain/event
title: "Event"
description: "What an event records, what to put in it, how it's stored, and how the rest of the model consumes it."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

An event records something that happened: `AccountOpened`, `MoneyDeposited`, `MoneyWithdrawn`. It’s named in the past tense because, by the time anyone sees it, it’s a fact.

Events are the only thing an aggregate stores as its history. Its state, every view, and every policy’s reaction are all derived from them.

## Events are the business’s memory

Businesses ran on events long before software. Take a bank’s ledger. It’s a list of things that happened, written in order and never erased: opened, deposited 100, withdrew 30. The balance at the bottom of the page is a sum of the lines above it. When the bank finds a mistake, it doesn’t rub out a line; it writes a new one that corrects it.

An event-sourced model keeps its history the same way. Each event is one line in the ledger of one aggregate instance. Everything else, the account’s state, its statement, the customer’s dashboard, is worked out from those lines. Because nothing is overwritten, you can always answer “how did it get like this?”, and a question nobody thought to ask last year can be answered from history you already have.

<svg viewBox="0 0 775 232" style="max-width: 891px" role="img" aria-label="The events are the record. The balance is what they add up to."><defs><marker id="alc-dag-arrow-yiecm8" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 213 56 L 287 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-yiecm8)"></path></g><g><path d="M 482 56 L 556 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-yiecm8)"></path></g><g><path d="M 656.5 88 L 656.5 138" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-yiecm8)"></path></g><g transform="translate(24, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 176 L 189 13 V 60 Q 189 64 185 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 176 0 V 13 H 189"></path><g transform="translate(33.800000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="53.800000000000004" y="28" dominant-baseline="middle">AccountOpened</text> <text fill="currentColor" x="94.5" y="46" text-anchor="middle" dominant-baseline="middle">seq 1</text></g></g> <g transform="translate(293, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 176 L 189 13 V 60 Q 189 64 185 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 176 0 V 13 H 189"></path><g transform="translate(14.299999999999997, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="34.3" y="28" dominant-baseline="middle">MoneyDeposited 100</text> <text fill="currentColor" x="94.5" y="46" text-anchor="middle" dominant-baseline="middle">seq 2</text></g></g> <g transform="translate(562, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 176 L 189 13 V 60 Q 189 64 185 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 176 0 V 13 H 189"></path><g transform="translate(18.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="38.2" y="28" dominant-baseline="middle">MoneyWithdrawn 30</text> <text fill="currentColor" x="94.5" y="46" text-anchor="middle" dominant-baseline="middle">seq 3</text></g></g> <g transform="translate(562, 144)"><rect stroke="currentColor" fill="none" width="189" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="94.5" y="28" text-anchor="middle" dominant-baseline="middle">balance: 70</text> <text fill="currentColor" x="94.5" y="46" text-anchor="middle" dominant-baseline="middle">derived</text></g></svg> <svg viewBox="0 0 410 324" style="max-width: 471px" role="img" aria-label="The events are the record. The balance is what they add up to."><defs><marker id="alc-dag-arrow-d8ybam" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 100.5 74 L 100.5 124" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-d8ybam)"></path></g><g><path d="M 100.5 194 L 100.5 244" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-d8ybam)"></path></g><g><path d="M 191 282 L 213 282" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-d8ybam)"></path></g><g transform="translate(10, 10)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 168 L 181 13 V 60 Q 181 64 177 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 168 0 V 13 H 181"></path><g transform="translate(29.800000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="49.800000000000004" y="28" dominant-baseline="middle">AccountOpened</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">seq 1</text></g></g> <g transform="translate(10, 130)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 168 L 181 13 V 60 Q 181 64 177 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 168 0 V 13 H 181"></path><g transform="translate(10.299999999999997, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="30.299999999999997" y="28" dominant-baseline="middle">MoneyDeposited 100</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">seq 2</text></g></g> <g transform="translate(10, 250)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 168 L 181 13 V 60 Q 181 64 177 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 168 0 V 13 H 181"></path><g transform="translate(14.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="34.2" y="28" dominant-baseline="middle">MoneyWithdrawn 30</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">seq 3</text></g></g> <g transform="translate(219, 250)"><rect stroke="currentColor" fill="none" width="181" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="90.5" y="28" text-anchor="middle" dominant-baseline="middle">balance: 70</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">derived</text></g></svg>

Event

The events are the record. The balance is what they add up to.

## Find the events

To find a business’s events, ask its people to tell the story of their work as a series of things that happened, in order. At a bank, it might go: “The customer registers. Their main account is opened. They deposit their pay. They withdraw cash. Late at night they withdraw again, and the account is frozen for review.” Each sentence in the past tense is an event: `CustomerRegistered`, `AccountOpened`, `MoneyDeposited`, `MoneyWithdrawn`, `AccountFrozen`.

Telling it as a story surfaces the things people care about, in their own words, and in the order they happen. The commands fall out next: for each event, ask what someone asked for to make it happen. The aggregates come after that: for each command, ask what has to be checked before it’s allowed, and what state that check needs.

## Name what the business saw

Name an event after what happened in the business, not after what changed in a database. `AccountFrozen` tells you something. `AccountUpdated` or `StatusChanged` tells you a row was written, and leaves you to dig through its fields to find out why.

A specific name is what lets the rest of the model react. A fraud policy listens for `MoneyWithdrawn`; a statement prints a line for each `MoneyDeposited`. Neither could subscribe to “the account changed” without re-deciding what kind of change it was.

One command can record more than one event when more than one thing happened. And a rejection is not an event: when a command is refused, nothing happened, so nothing is recorded.

## Declare an event

An event has a name and, usually, some data.

```typescript
export class MoneyWithdrawn extends Domain.Event("MoneyWithdrawn", {
  amount: Schema.Int,
  balanceAfter: Schema.Int,
}) {}
```

Each field is a schema, so events are encoded the same way wherever they’re stored or sent. An event that needs no data has no fields: `Domain.Event("AccountClosed")`.

## Describe it

An event’s last argument is its [annotations](annotations.md). Give it a `description` that says what happened, in the business’s words. It is the event Schema’s description too.

```typescript
export class MoneyWithdrawn extends Domain.Event(
  "MoneyWithdrawn",
  { amount: Schema.Int, balanceAfter: Schema.Int },
  { description: "Money left an account. Amounts are in cents." },
) {}
```

## Only decide creates events

Events are created in exactly one place: an aggregate’s `decide` handler, as its answer to a command.

```typescript
: [new MoneyWithdrawn({ amount: cmd.amount, balanceAfter: s.balance - cmd.amount })],
```

The command declares which events it may record, so `decide` can’t return anything else. Once they’re committed, they’re never changed or removed. A mistake is corrected the way an accountant corrects one, with a new event that reverses it, so the history still shows both.

## Record what readers will need

Put in the event what someone reading it later needs to understand it on its own, without asking the aggregate what its state was at the time.

`MoneyWithdrawn` carries `balanceAfter` for that reason. The aggregate knows the balance when it decides, so it’s free to record. A view that shows the balance, a statement that prints it on each line, and a policy that flags low balances can all read it straight from the event. Without it, each would have to keep its own running total and hope it never drifts.

The same goes for who did something and why. When a support agent freezes an account, the event records the agent and the reason, so the history explains itself.

```typescript
export class AccountFrozen extends Domain.Event("AccountFrozen", {
  reason: Schema.String,
  by: Actor,
}) {}
```

## Keep old events readable

Events are kept for good, so every event you’ve ever recorded will be read again: each time an aggregate rebuilds its state, and each time a view is built from history. Treat an event’s shape as a promise to your future self.

Add a new field as optional, so events recorded before it existed still decode. When the meaning of something changes, add a new event rather than changing what an old one means. A history in which `MoneyWithdrawn` meant one thing in March and another in May is a history nobody can read.

```typescript
export class MoneyWithdrawn extends Domain.Event("MoneyWithdrawn", {
  amount: Schema.Int,
  balanceAfter: Schema.Int,
  channel: Schema.optional(Schema.Literals(["atm", "app"])), // added later
}) {}
```

## Fold events into state

The aggregate turns each event into its next state with `evolve`. This is where an event’s data takes effect.

```typescript
evolve: {
  AccountOpened: (_, e) => ({ _tag: "Open" as const, owner: e.owner, balance: 0 }),
  MoneyDeposited: (s, e) => (s._tag === "Open" ? { ...s, balance: e.balanceAfter } : s),
  MoneyWithdrawn: (s, e) => (s._tag === "Open" ? { ...s, balance: e.balanceAfter } : s),
},
```

`evolve` can’t refuse an event. It already happened, and only `decide` gets to say no. `decide` returns a typed rejection such as `NotOpen` before it records a deposit to an account that isn’t open, so `evolve` only has to cover every state. See [Aggregate](aggregate.md#apply-events-with-evolve).

## The envelope

When an event commits, it’s stored inside an envelope that says where and when it happened. `id` is the aggregate instance and `stream` names its history, like `Account/a-1`. `seq` is the event’s position in that stream, starting at 1. `at` is the commit time, and `commandId` is the id of the command that caused it.

`evolve` receives the envelope as its third argument, for state that depends on when something happened.

```typescript
MoneyWithdrawn: (_, e, { at }) => ({
  balance: e.balanceAfter,
  lastWithdrawal: DateTime.formatIsoDateUtc(at),
}),
```

The `commandId` lets a client recognize the events its own commands caused, which is how an app confirms an [optimistic update](optimistic-updates.md).

## Delivered to views and policies

In the same commit that stores an event, the instance records a delivery for every [view](view.md) and [policy](policy.md) that consumes it. After the commit, each delivery is sent. Every consumer receives every event it consumes at least once, and in the order the instance recorded them.

<svg viewBox="0 0 456 296" style="max-width: 524px" role="img" aria-label="One event, delivered to every view that reads it."><defs><marker id="alc-dag-arrow-7dbe57" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 148 C 225 148, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-7dbe57)"></path></g><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-7dbe57)"></path></g><g><path d="M 188 148 C 225 148, 225 240, 262 240" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-7dbe57)"></path></g><g transform="translate(24, 116)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">MoneyWithdrawn</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Account a-1</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(36.9, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="56.9" y="28" dominant-baseline="middle">Statement</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g> <g transform="translate(268, 208)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">DailyOutflow</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g></svg> <svg viewBox="0 0 348 268" style="max-width: 400px" role="img" aria-label="One event, delivered to every view that reads it."><defs><marker id="alc-dag-arrow-1tizos" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 160 134 C 171 134, 171 42, 182 42" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-1tizos)"></path></g><g><path d="M 160 134 C 171 134, 171 134, 182 134" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-1tizos)"></path></g><g><path d="M 160 134 C 171 134, 171 226, 182 226" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-1tizos)"></path></g><g transform="translate(10, 102)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 137 L 150 13 V 60 Q 150 64 146 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 137 0 V 13 H 150"></path><g transform="translate(10.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="30.400000000000006" y="28" dominant-baseline="middle">MoneyWithdrawn</text> <text fill="currentColor" x="75" y="46" text-anchor="middle" dominant-baseline="middle">Account a-1</text></g></g> <g transform="translate(188, 10)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="150" height="64" rx="32"></rect><g transform="translate(10.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="30.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="75" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g> <g transform="translate(188, 102)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="150" height="64" rx="32"></rect><g transform="translate(29.9, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="49.9" y="28" dominant-baseline="middle">Statement</text> <text fill="currentColor" x="75" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g> <g transform="translate(188, 194)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="150" height="64" rx="32"></rect><g transform="translate(18.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="38.2" y="28" dominant-baseline="middle">DailyOutflow</text> <text fill="currentColor" x="75" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g></svg>

EventView

One event, delivered to every view that reads it.

A view can record events of its own as it folds, such as one `StatementLine` per deposit or withdrawal. Those are delivered the same way, so another view or policy can build on them.

## Follow an instance’s events

A client reads an account’s history through a view that follows it, then keeps receiving new events as they’re recorded. `AccountSummary` declares no events of its own, so it emits every event it applies.

```typescript
export const followAccount = Effect.fn(function* (bank: CustomerClient, accountId: AccountId) {
  yield* bank.AccountSummary.events(accountId).pipe(
    Stream.filter(Domain.Read.isEvent),
    Stream.runForEach(({ event, envelope }) =>
      Effect.log(\`#${envelope.seq} ${event._tag} (command ${envelope.commandId})\`),
    ),
  );
});
```

`events` replays from the beginning, sends a `CaughtUp` frame, then continues live. `Domain.Read.isEvent` keeps only the events. Pass `after` with a `seq` to resume from a position you’ve already seen.

Next, [Aggregate](aggregate.md): where commands are decided and events are recorded.
