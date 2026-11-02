---
url: https://alchemy.run/domain/command
title: "Command"
description: "How a command names its target, carries its data, declares every outcome, and reaches its aggregate exactly once."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A command asks one aggregate instance for a change. It’s a request, so it can be refused, and it’s named as an instruction: `OpenAccount`, `DepositMoney`, `WithdrawMoney`.

A command is also a contract. Before you send one, its declaration tells you which instance it goes to, what data it carries, which events it may record, which refusals it may return, and what you get back. This chapter starts with how to find the commands in a business, then goes through the declaration one part at a time, and follows a command as it’s sent.

## Find the commands in your business

Commands come from the verbs people use when they describe their work. At a bank, for example, a customer *opens* an account, *deposits* money, *withdraws* cash, *requests* a transfer; a support agent *freezes* an account, *approves* a large transfer. Each of those sentences is a command, named the way the business says it.

<svg viewBox="0 0 700 204" role="img" aria-label="Each command is one thing someone wants, in the words they'd use to ask for it."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Customer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">who asks</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">Support agent</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">who asks</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">WithdrawMoney</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a customer's intent</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">FreezeAccount</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">an agent's intent</text></g></g> <g transform="translate(512, 70)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">decides both</text></g></g></svg>

CommandAggregateClient

Each command is one thing someone wants, in the words they'd use to ask for it.

The opposite is a command named after the data it touches: `UpdateAccount`, `SetBalance`, `PatchCustomer`. Those say what changes in storage, but not why. An `UpdateAccount` that sets `frozen: true` and one that sets `owner: "kim"` are different requests with different rules: who may make them, when they’re refused, and what the history should say afterwards. Give each intent its own command, `FreezeAccount` and `ChangeOwner`, so each gets its own rules, its own refusals and its own event.

A useful test: read the command’s name aloud to someone who works in the business. If they’d say “yes, customers do that”, it’s a good command. If they’d ask “update it how?”, split it.

## Carry the intent, not the outcome

A command carries what the caller knows and wants. It doesn’t carry what the aggregate will work out. A withdrawal says how much to withdraw; it doesn’t say what the balance will be afterwards, because only the account knows its balance at the moment it decides.

```typescript
new WithdrawMoney({ amount: 30 }); // the intent
new MoneyWithdrawn({ amount: 30, balanceAfter: 70 }); // the outcome, decided by the account
```

That split keeps every rule in the aggregate. A client can’t send a balance it made up, and two clients that both read a balance of 100 can’t both withdraw 80: the account decides each withdrawal against its own state, one at a time.

## Declare a command

Here is a withdrawal from the bank in the [tutorial](tutorial/part-1.md).

```typescript
export class WithdrawMoney extends Domain.Command("WithdrawMoney", {
  target: AccountId,
  input: { amount: Schema.Int },
  events: [MoneyWithdrawn],
  rejects: [NotOpen, InsufficientFunds],
  reply: { balance: Schema.Int },
}) {}
```

The first argument is the command’s name. It’s how the command is identified on the wire and in the aggregate’s handlers, so it must be unique within a model.

## Keep it with its events and rejections

A command, the events it records and the rejections it returns are one unit: you read them together, and change them together. Put them in one file named after the command, with a `.command.ts` suffix, like `WithdrawMoney.command.ts`. See [Project layout](project-layout.md).

```typescript
export class MoneyWithdrawn extends Domain.Event("MoneyWithdrawn", {
  amount: Schema.Int,
  balanceAfter: Schema.Int,
}) {}

export class InsufficientFunds extends Domain.Rejection("InsufficientFunds", {
  balance: Schema.Int,
  requested: Schema.Int,
}) {}
```

When more than one command uses something, it moves to a file of its own. Deposits and withdrawals both refuse an account that isn’t open, so `NotOpen` has its own file, `NotOpen.rejection.ts`, and both import it.

```typescript
export class NotOpen extends Domain.Rejection("NotOpen") {}
```

The aggregate’s file then holds only its state and how it decides and evolves, and imports each command from its file. The same goes for ids, like `AccountId`, which the commands and the aggregate share.

## Address one instance

`target` is the schema of the id the command is addressed to. A withdrawal goes to one account, so its target is an `AccountId`.

```typescript
export const AccountId = Schema.String.pipe(Schema.brand("AccountId"));
```

The brand makes the id its own type. A `CustomerId` is a string too, but passing one where an `AccountId` is expected is a type error, so a command can’t be sent to the wrong kind of instance.

The target must be the id of the aggregate that lists the command. If `Account` lists a command whose target is a `CustomerId`, or a plain `Schema.String`, that entry in `Account` ’s `commands` is a type error that names the command.

The target isn’t part of the command’s data. You pass it alongside the command when you send it, and the model uses it to route the command to that one instance.

## Carry the request’s data

`input` lists the fields the caller provides, each as a schema. You create the command with them, and the aggregate reads them in `decide`.

```typescript
WithdrawMoney: (s, cmd) =>
  s._tag !== "Open"
    ? new NotOpen()
    : cmd.amount > s.balance
      ? new InsufficientFunds({ balance: s.balance, requested: cmd.amount })
      : [new MoneyWithdrawn({ amount: cmd.amount, balanceAfter: s.balance - cmd.amount })],
```

The schemas also decode the command when it arrives from a client, so `decide` only ever sees well-formed input. A command with no input, like a request to close an account, leaves `input` out.

## Declare every outcome

A command can end in one of two ways: the aggregate records events, or it refuses. `events` lists the events the command may record, and `rejects` lists the refusals it may return.

A refusal is a `Domain.Rejection`, a typed error that can carry data about why.

```typescript
export class InsufficientFunds extends Domain.Rejection("InsufficientFunds", {
  balance: Schema.Int,
  requested: Schema.Int,
}) {}
```

Both lists are enforced by the types. The aggregate’s `decide` handler for `WithdrawMoney` can only return `MoneyWithdrawn` events or one of its two rejections. And whoever sends it gets exactly `NotOpen | InsufficientFunds` in the error channel, so they can handle each case by name.

A rejection means nothing happened. No event is recorded and the state is unchanged, so the caller can safely try again with different input.

Name a rejection after the business reason, the way you’d explain it to the customer: `InsufficientFunds`, `NotOpen`, `DailyLimitExceeded`. Give it the data the caller needs to do something about it. `InsufficientFunds` carries the balance and the amount requested, so an app can say “you have 50, you asked for 80” without a second read.

## Annotate a rejection

`Domain.Rejection` is `Schema.TaggedError` with its own type filled in, so its third argument takes the same annotations. An HTTP API reads `httpApiStatus` from it, as it does for any Effect error:

```typescript
export class NotOwner extends Domain.Rejection(
  "NotOwner",
  {},
  { httpApiStatus: 403, description: "Only the account's owner may do this." },
) {}
```

`rejects` takes any tagged error class, so a `Schema.TaggedError` of your own, or one of Effect’s `HttpApiError` classes, works as a rejection too:

```typescript
class WithdrawMoney extends Domain.Command("WithdrawMoney", {
  target: AccountId,
  input: { amount: Cents },
  events: [MoneyWithdrawn],
  rejects: [InsufficientFunds, HttpApiError.Forbidden],
}) {}
```

## Report a fact that can’t be refused

Some commands don’t ask permission. They tell the model about something that already happened outside it: a card payment settled at the payment processor, a deadline passed. The model can’t refuse a fact, so these commands declare no `rejects`. Their aggregate records what the fact means for it instead, even when the news is bad.

```typescript
export class RecordSettlement extends Domain.Command("RecordSettlement", {
  target: AccountId,
  input: { settlementId: Schema.String, amount: Cents },
  events: [SettlementReceived, SettlementOnClosedAccount],
}) {}
```

Money that settles on a closed account is still recorded, as `SettlementOnClosedAccount`, and a policy refunds it. See [External System](external-system.md#turn-outside-events-into-commands).

## Reply to the caller

`reply` is what the caller gets back after the events commit. Here, the balance after the withdrawal.

The aggregate computes it from its state after the events, in a `reply` handler named after the command.

```typescript
reply: {
  DepositMoney: (s) => ({ balance: s._tag === "Open" ? s.balance : 0 }),
  WithdrawMoney: (s) => ({ balance: s._tag === "Open" ? s.balance : 0 }),
},
```

A command without a `reply` still tells the caller where it committed. See [the receipt](#send-it) below.

## Guard it with middleware

Before a command reaches its aggregate, it can pass through middleware. In the full example bank, a withdrawal first authenticates the caller with `Session`, then asks for a second factor with `MfaChallenge`.

```typescript
export class WithdrawMoney extends Domain.Command("WithdrawMoney", {
  target: AccountId,
  middleware: [Session, MfaChallenge],
  input: { amount: Cents },
  events: [MoneyWithdrawn],
  rejects: [NotOpen, NotOwner, Frozen, InsufficientFunds, DailyLimitExceeded],
  reply: { balance: Cents },
  // Recorded on every span the withdrawal passes through; nothing else from it is.
  attributes: (cmd) => ({ "bank.amount": cmd.amount }),
}) {}
```

`attributes` chooses what its spans record; see [Observability](observability.md).

Middleware runs in order, and any of it can stop the command with its own error, like `Unauthorized`. What it provides, such as who the caller is, travels with the command to the aggregate, where `decide` reads it to authorize the change. See [Sessions and middleware](sessions.md).

A client can run the same `decide` with `Account.predict` when it provides what the middleware would, from a client-side Layer. See [Optimistic updates](optimistic-updates.md#bound-this-pattern).

## Send it

Inside a hosted model, you send a command through the aggregate’s client, with the target id first.

```typescript
yield* accounts
  .send(AccountId.make(\`${source.id}-main\`), new OpenAccount({ owner: source.id }))
  .pipe(Effect.catchTag("AlreadyOpen", () => Effect.void));
```

From a client of an [Api](api.md), each command is a method that takes the target, the input, and optional call options: `bank.WithdrawMoney(accountId, { amount: 2_000 })`.

Either way, `send` succeeds once the events have committed, with a receipt. It holds the stream and the version the instance committed at, the events it recorded, and the reply.

## Send it exactly once

Networks fail after a command commits but before its receipt arrives. A client that retries could withdraw twice.

Every command has an id to prevent that. The instance stores each receipt under its command’s id, and when the same id arrives again, it returns the stored receipt without deciding again. Pass your own `commandId` when you might retry:

```typescript
const first = yield* bank.WithdrawMoney(accountId, { amount: 2_000 }, { commandId: requestId, ...mfa });
const retry = yield* bank.WithdrawMoney(accountId, { amount: 2_000 }, { commandId: requestId, ...mfa });
first.version === retry.version; // true
```

Both calls return the same receipt, and the money leaves the account once. Without a `commandId`, each send gets a fresh one.

Commands sent by a [policy](policy.md) get ids derived from the event that caused them, so a policy that runs twice for the same event sends the same commands, and each takes effect once.

## Send it later

A command can also be scheduled to arrive at a later time, which is how deadlines work: “expire this transfer if nobody approves it within a day”. Only commands that can’t be refused can be scheduled, because there’s no caller waiting to handle a rejection. See [Deadlines](deadlines.md).

```typescript
yield* transfers.schedule(source, new ExpireTransfer(), { after: APPROVAL_WINDOW }).pipe(
  Effect.catchTag("AlreadyExecuted", () => Effect.void),
);
```

Scheduling fails with `AlreadyExecuted` when the transfer already handled a command with that id.

Next, [Event](event.md): what a command records when it succeeds.
