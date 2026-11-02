---
url: https://alchemy.run/domain/aggregate
title: "Aggregate"
description: "How an aggregate holds state, decides commands, applies events, and keeps its rules consistent."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

An aggregate is the part of a model that makes decisions. Each instance, one bank account for example, holds the state its rules need, takes one command at a time, and answers with events or a refusal.

This chapter starts with what an aggregate is for and how to find one, then builds the account from the [tutorial](tutorial/part-1.md) piece by piece: its state, how it decides, how it applies what it decided, and what makes it safe under concurrent use. It ends with how to draw an aggregate’s boundary.

## What an aggregate protects

Every business has rules that must never be broken, not even for a moment. An account’s balance never goes below zero. A username belongs to at most one customer. A transfer is either approved or expired, never both. DDD calls these **invariants**, and an aggregate is the thing that keeps one true.

To find the invariants, ask the business what must never happen. Then ask which commands could make it happen. A withdrawal could overdraw an account, so `WithdrawMoney` has to be decided by something that knows the balance. That something is the `Account` aggregate: it holds the balance, and every command that could break the rule about it goes through it.

<svg viewBox="0 0 456 204" role="img" aria-label="Every command that touches the balance goes through the one thing that guards it."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">DepositMoney</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">changes the balance</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">WithdrawMoney</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">could overdraw</text></g></g> <g transform="translate(268, 70)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">balance ≥ 0</text></g></g></svg>

CommandAggregate

Every command that touches the balance goes through the one thing that guards it.

So an aggregate isn’t a table or an entity with getters and setters. It’s a rule, together with exactly the state the rule needs, and the commands that have to respect it. Find the rule first; the state and the commands follow.

## Give it an identity

An aggregate type has many instances, and each is addressed by an id. The account’s id is an `AccountId`, the same schema its commands use as their `target`.

```typescript
export class Account extends Domain.Aggregate("Account", {
  id: AccountId,
```

Instances aren’t created explicitly. Every id exists from the start, in the aggregate’s initial state, and a command sent to a new id is decided against that state. Opening an account is just the first command an account accepts.

## Model the state its rules need

The state holds what `decide` needs to check its rules, and nothing more. An account must know whether it’s open, who owns it, and its balance.

```typescript
const State = Schema.Union([
  Schema.TaggedStruct("Unopened", {}),
  Schema.TaggedStruct("Open", { owner: Schema.String, balance: Schema.Int }),
]);
```

A union of tagged states makes the lifecycle explicit. An `Unopened` account has no owner and no balance, so there’s no field to forget to check. Code that wants the balance has to establish that the account is `Open` first, and the types enforce it.

```typescript
state: State,
initial: { _tag: "Unopened" },
```

`initial` is the state of every instance before its first event.

Keep the statement, the transaction count and the monthly totals out of here. The account doesn’t need them to decide anything, so they belong in [views](view.md).

## Decide each command

`decide` has one handler per command in `commands`. A handler receives the current state and the command, and returns the events to record or a rejection.

```typescript
WithdrawMoney: (s, cmd) =>
  s._tag !== "Open"
    ? new NotOpen()
    : cmd.amount > s.balance
      ? new InsufficientFunds({ balance: s.balance, requested: cmd.amount })
      : [new MoneyWithdrawn({ amount: cmd.amount, balanceAfter: s.balance - cmd.amount })],
```

Read it as the business rule it is: an account that isn’t open can’t be withdrawn from; a withdrawal can’t exceed the balance; otherwise, the money is withdrawn and here’s the new balance.

`decide` doesn’t change the state. It only describes what should happen. That keeps every rule in a function you can test with a state and a command, and nothing else.

A handler can also return an empty array. The command is accepted, but there’s nothing to record, like freezing an account that’s already frozen. The caller still gets a receipt.

So every handler picks one of three answers. Record events when the command changes something. Return `[]` when the state already is what the command asks for. Return a rejection when the command can’t be done, so the caller sees why in its type.

## Name the rules decide shares

Most commands check the same thing first: the account must be open. Write that check once, as a pure function that returns the narrowed state or the rejection.

```typescript
type Open = Extract<State, { _tag: "Open" }>;

const openFor = (s: State): Open | NotOpen => (s._tag === "Open" ? s : new NotOpen());
```

Each handler starts from it. After the check, `o` is an `Open` account, so the rest of the rule reads its balance without another test.

```typescript
DepositMoney: (s, cmd) => {
  const o = openFor(s);
  if (o._tag !== "Open") return o;
  return [new MoneyDeposited({ amount: cmd.amount, balanceAfter: o.balance + cmd.amount })];
},
```

A handler written with `Effect.fn` fails with the rejection instead: `if (o._tag !== "Open") return yield* o;`.

## Record the facts evolve needs

An event carries the outcome `decide` worked out, not just the request. `MoneyDeposited` records `balanceAfter`, so `evolve` copies it and never repeats the arithmetic.

```typescript
export class MoneyDeposited extends Domain.Event("MoneyDeposited", {
  amount: Schema.Int,
  balanceAfter: Schema.Int,
}) {}
```

The rules then live in one place. A view or a policy reading the event sees the same balance the account does.

## Apply events with evolve

`evolve` has one handler per event. It receives the current state and the event, and returns the next state.

```typescript
evolve: {
  AccountOpened: (_, e) => ({ _tag: "Open" as const, owner: e.owner, balance: 0 }),
  MoneyDeposited: (s, e) => (s._tag === "Open" ? { ...s, balance: e.balanceAfter } : s),
  MoneyWithdrawn: (s, e) => (s._tag === "Open" ? { ...s, balance: e.balanceAfter } : s),
},
```

Keeping `evolve` separate from `decide` is what lets the state be derived from events. `decide` runs once, when the command arrives, and may refuse. `evolve` must always accept, because it applies events that have already been recorded, and applying the same events must always give the same state.

`evolve` never fails either, and never throws. Refusing is `decide` ’s job: every command that records `MoneyDeposited` has already returned `NotOpen` for an account that isn’t open. So each handler covers every state, and keeps a state its event can’t follow. An event that moves the account to another stage, like `AccountOpened`, returns the new state whole.

## Reply from the new state

A command that declares a `reply` gets one from a `reply` handler. It runs after `evolve`, on the state the events produced.

```typescript
reply: {
  DepositMoney: (s) => ({ balance: s._tag === "Open" ? s.balance : 0 }),
  WithdrawMoney: (s) => ({ balance: s._tag === "Open" ? s.balance : 0 }),
},
```

Like `evolve`, `reply` covers every state and never fails. The reply is stored with the events, so a retried command returns the same reply as the first time.

## Focus on a state with optics

Once many handlers repeat the same `s._tag === "Open" ? ... : s`, an optic from `effect/Optic` can say it once. It focuses on the `Open` state, and leaves any other state unchanged.

```typescript
import * as Optic from "effect/Optic";

const open = Optic.id<State>().tag("Open");
const balance = open.key("balance");
```

A handler that sets one field replaces it through the optic.

```typescript
MoneyDeposited: (s, e) => Optic.replace(s, balance, e.balanceAfter),
```

A handler that changes several fields modifies the focused state.

```typescript
OwnerChanged: (s, e) => open.modify((o) => ({ ...o, owner: e.owner }))(s),
```

Optics are optional. They give the same states as the plain checks, with less repetition. The [example bank](https://github.com/alchemy-run/alchemy/tree/main/examples/cloudflare-domain) uses them for its account.

## Never refuse a compensation

Some commands undo a step another process already took, like refunding a transfer whose credit failed. Refusing one would strand the money, so its handler records a fact for every state instead of rejecting.

```typescript
RefundTransfer: (s, cmd) =>
  s._tag === "Open"
    ? [
        new TransferRefunded({
          transferId: cmd.transferId,
          amount: cmd.amount,
          balanceAfter: s.balance + cmd.amount,
        }),
      ]
    : [new RefundStranded({ transferId: cmd.transferId, amount: cmd.amount })],
```

`RefundStranded` tells a policy or a person that the account closed before the money came back. The [saga](saga.md) that sent the refund never has to handle a refusal it can do nothing about.

## Use context in decisions

Some rules need more than the state and the command. Only the account’s owner may withdraw from it, so `decide` needs to know who’s asking.

A command’s [middleware](sessions.md) can provide that. When it does, the handler can be an `Effect.fn` and read it. In the example bank, the withdrawal handler first checks that the current actor may act on the account.

```typescript
const authorized = Effect.fn(function* (s: AccountState) {
  const o = openFor(s, yield* CurrentActor);
  if (o._tag !== "Open") return yield* o;
  return o;
});
```

Rules can depend on time, too. The handler’s third argument carries `now`, the time the command is decided. The example bank uses it for a daily withdrawal limit:

```typescript
const used = o.today.day === dayOf(now) ? o.today.withdrawn : 0;
if (used + cmd.amount > DAILY_LIMIT)
  return yield* new DailyLimitExceeded({ remaining: DAILY_LIMIT - used });
```

`now` is the time the command is stamped with, and its events record the same time as `at` in their envelopes, which is what `evolve` uses to track the day’s total. A [story](story.md) can move the clock, so rules like this are testable.

## One command at a time

Each instance processes one command at a time. When two withdrawals for the same account arrive together, the second is decided against the state the first one left. Both can’t pass the balance check on the same money.

<svg viewBox="0 0 700 204" role="img" aria-label="The second withdrawal is decided against the balance the first one left."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 102 C 469 102, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 102 C 469 102, 469 148, 506 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">WithdrawMoney 80</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">arrives first</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">WithdrawMoney 80</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">arrives second</text></g></g> <g transform="translate(268, 70)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">balance: 100</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">MoneyWithdrawn</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">balance: 20</text></g></g> <g transform="translate(512, 116)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(5.700000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="25.700000000000003" y="28" dominant-baseline="middle">InsufficientFunds</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">balance: 20</text></g></g></svg>

CommandAggregateEventRejection

The second withdrawal is decided against the balance the first one left.

Everything a command produces, its events, the new state, the reply and the deliveries to views and policies, commits together or not at all. This is what makes an instance a consistency boundary: its rules hold at every moment, for every observer.

Nothing spans two instances. A transfer between accounts can’t debit one and credit the other in a single step, because they’re separate boundaries. Instead, each account handles its own side, and a [saga](saga.md) drives the steps in order.

That shapes how big an aggregate should be. Draw it around the rules that must hold together, and no wider. A customer and their accounts are separate aggregates: no rule needs a customer’s name and an account’s balance to change in the same step, and keeping them apart means one busy account never slows down the others.

## Give a rule across instances its own aggregate

Some rules span many instances. Usernames must be unique across every customer, but each customer is its own aggregate, and no customer can see the others. Making `Customer` check every other customer would need a transaction across all of them, which doesn’t exist.

The answer is to ask what the rule is about. Uniqueness is about one username, so make the username the aggregate. `UsernameClaim` is keyed by the username itself, and holds who has it:

```typescript
export class UsernameClaim extends Domain.Aggregate("UsernameClaim", {
  id: Username,
  state: Schema.Union([
    Schema.TaggedStruct("Free", {}),
    Schema.TaggedStruct("Claimed", { customerId: CustomerId }),
  ]),
  initial: { _tag: "Free" },
  commands: [ClaimUsername],
  decide: {
    ClaimUsername: (s, cmd) =>
      s._tag === "Free"
        ? [new UsernameClaimed({ customerId: cmd.customerId })]
        : s.customerId === cmd.customerId
          ? []
          : new UsernameTaken(),
  },
  evolve: {
    UsernameClaimed: (_, e) => ({ _tag: "Claimed" as const, customerId: e.customerId }),
  },
}) {}
```

Two customers who ask for `sam` at the same moment both send `ClaimUsername` to the same instance, `UsernameClaim/sam`. It decides one at a time, so one wins and the other is told `UsernameTaken`. The rule holds without anything spanning two aggregates. See [Unique values](unique-values.md).

## Keep a process’s state in an aggregate

A process that takes time, like a transfer that may wait a day for approval, has state of its own: requested, awaiting approval, completed, failed. Rules apply to that state too. An approval that arrives after the deadline must be refused, and an expiry that arrives after an approval must do nothing.

Those rules make the process an aggregate. In the example bank, `Transfer` holds where each transfer stands, and both `ApproveTransfer` and `ExpireTransfer` go through it, so whichever arrives first wins:

```typescript
ExpireTransfer: (s, _, { id }) =>
  s._tag === "AwaitingApproval"
    ? [new TransferFailed({ transferId: id, reason: "not approved in time" })]
    : [],
```

The money itself still moves through the two accounts, each guarding its own balance. A [saga](saga.md) carries out the steps, and reports each outcome back to the `Transfer`, so there’s one place that knows how every transfer ended.

## Send it commands

Inside a hosted model, the aggregate is a service. `yield* Account` gives you its client, and `send` takes an instance id and a command.

```typescript
yield* accounts
  .send(AccountId.make(\`${source.id}-main\`), new OpenAccount({ owner: source.id }))
  .pipe(Effect.catchTag("AlreadyOpen", () => Effect.void));
```

The same client reads an instance’s current state with `accounts.state(id)`, schedules a command for later with `accounts.schedule(id, command, { after })`, which returns the command id and when it’s due, and cancels a pending one with `accounts.unschedule(id, commandId)`. Both fail with `AlreadyExecuted` when that command already ran, and `unschedule` fails with `AlreadyCancelled` or `NotScheduled` when there’s nothing pending to cancel. See [Deadlines](deadlines.md).

Clients outside the model send commands through an [Api](api.md). An aggregate with a `read` contract can also be read through one, so a client can fetch an account, watch it change, and follow its events.

Next, [View](view.md): the query side, built from the events aggregates record.
