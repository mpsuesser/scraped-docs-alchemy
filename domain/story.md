---
url: https://alchemy.run/domain/story
title: "Story"
description: "A specification you can run. Given some history, when a command is sent, then these events are recorded."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A story describes one piece of behavior in the business’s own words: given this history, when this command is sent, then these events are recorded. It’s also a test. It runs your real model in memory, so what it describes is what the model does.

This last chapter shows why rules are best written as examples, how to read and write a story, and how stories cover every building block in this manual: aggregates, views, policies, external systems and time.

## Agree on rules with examples

Ask someone who does the work how it works, and you’ll usually get a rule. At a bank: “you can’t take out more than you have”. Ask them what happens when an account holds 100 and the customer asks for 150, and you’ll get an example: “it’s refused, and we tell them they have 100”. The example is sharper. It has numbers, a starting point and an outcome, and it settles questions the rule leaves open, like what the refusal should say.

DDD teams use examples like that to pin down a rule with the business before writing it, and to keep it pinned down afterwards. This is sometimes called **specification by example**. A story is that example, written in the model’s language, and run against the model.

<svg viewBox="0 0 944 232" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 56 L 750 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 594 88 L 594 176 L 750 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">AccountOpened</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">given · the history</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">WithdrawMoney</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">when · a command</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">decides</text></g></g> <g transform="translate(756, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">MoneyWithdrawn</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">then · the events</text></g></g> <g transform="translate(756, 144)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(5.700000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="25.700000000000003" y="28" dominant-baseline="middle">InsufficientFunds</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">or · the rejection</text></g></g></svg>

CommandAggregateEventRejection

## Given, when, then

Every story has the same three parts, because every change in an event-sourced model has the same three parts.

**Given** is the history: the events that have already happened. It’s the starting point, stated as facts, not as a sequence of commands you’d have to send and hope succeed. **When** is the command someone sends. **Then** is what the model records in answer: the events, or the rejection when it refuses.

Read one aloud and it sounds like the business. This story says an overdraft is refused, and what the refusal reports:

```typescript
it.effect("an overdraft is rejected with the evidence", () =>
  story(
    Story.given(account, opened, new MoneyDeposited({ amount: 50, balanceAfter: 50 })),
    asSam,
    Story.when(account, new WithdrawMoney({ amount: 80 })),
    Story.rejected(new InsufficientFunds({ balance: 50, requested: 80 })),
  ),
);
```

“Given an account opened with 50 in it, when Sam withdraws 80, it’s refused because the balance is 50 and 80 was asked for.” `asSam` says who’s asking, as the command’s middleware would in production.

## Write the history as events

`given` writes events straight onto the account. It doesn’t send `OpenAccount` and `DepositMoney` to get there. That keeps a story about the rule it’s testing: an overdraft story shouldn’t fail because the rules for opening an account changed.

It also lets a story start from any point in an account’s life, including ones that are tedious to reach through commands, like an account opened two years ago with a hundred transactions. You state the facts that matter to the rule, and nothing else.

## Check what’s recorded and replied

When the command succeeds, `then` checks the events it recorded, and `replied` checks what the caller got back:

```typescript
Story.when(account, new WithdrawMoney({ amount: 30 })),
Story.then(account, new MoneyWithdrawn({ amount: 30, balanceAfter: 70 })),
Story.replied({ balance: 70 }),
```

The checks after a `when` are type-checked against its command. `rejected` takes only a rejection the command declares, and `replied` only its reply type, so a story can’t check for an outcome the command can’t have.

## Check what clients see

A story runs the views too, so it can check what a client would read after the command. `view` checks a view’s state for one key, and `emitted` checks the events a view has emitted:

```typescript
Story.view(AccountSummary, account, { balance: 70 }),
Story.emitted(Statement, account, [
  { kind: "deposit", amount: 100, balance: 100 },
  { kind: "withdrawal", amount: 30, balance: 70 },
]),
```

## Follow a command’s consequences

A command rarely ends at its aggregate. A large withdrawal pushes the day’s total past the fraud threshold, the fraud policy asks the scoring service, and a high score freezes the account. A story follows all of it, in order:

```typescript
Story.when(account, new WithdrawMoney({ amount: 60_000 })),
Story.then(account, new MoneyWithdrawn({ amount: 60_000, balanceAfter: 30_000 })),
Story.call(FraudCheck.score, { accountId: account.id, amount: 60_000 }),
Story.succeed(FraudCheck.score, { risk: 0.95 }),
Story.then(account, new AccountFrozen({ reason: "fraud risk 0.95", by: system("FraudReview") })),
```

The call to the fraud service waits until the story answers it. `call` checks what the policy asked, and `succeed` gives the answer. Answer `0.1` instead, with no freeze after it, and you have the story for a withdrawal that isn’t risky. Every answer the outside world could give is one line away. See [External System](external-system.md#test-external-system-calls).

## Move the clock

Some rules depend on time: a daily withdrawal limit, a transfer that expires if nobody approves it within a day. A story has its own clock, which only moves when the story moves it:

```typescript
Story.clock("2026-01-02T08:59:00Z"),
Story.then(transfer),
Story.clock("2026-01-02T09:00:00Z"),
Story.then(transfer, new TransferFailed({ transferId: transfer.id, reason: "not approved in time" })),
```

A minute before the deadline, nothing has happened: `then` with no events checks that. At the deadline, the scheduled expiry fires and the transfer fails. See [Deadlines](deadlines.md).

## It runs the whole model

`Story.make` builds a runner from your model, so a story runs everything a command sets off. The aggregate decides, views update, policies react and send their own commands, and sagas run their steps.

```typescript
const story = Story.make(Bank, {
  layer: BankHandlers.pipe(Layer.provide(MemoryLedgerStore)),
  stub: [FraudCheck, Payments],
});
```

Everything runs in memory, so a story that spans several aggregates, a policy and a saga still runs in milliseconds. There’s no database to reset and nothing to mock: the history is the setup, and the events are the assertions.

## A passing story accounts for everything

A story fails if a command records an event and no `then` checks it, if a command is rejected and no step checks the rejection, or if an external system call is still waiting at the end. This includes the events a policy or saga records on other instances.

That’s what makes a passing story trustworthy. It can’t pass while a policy quietly freezes an account nobody mentioned, or while a refund goes out that nobody expected. Every outcome of the command is written down in the story, so the story is a complete description of what happens.

## Write one story per rule

Name each story after the rule it pins down, as a sentence the business would agree with: “an overdraft is rejected with the evidence”, “a risky withdrawal freezes the account”. The list of story names in a test file then reads as the rules of that part of the business.

For each command, write a story for the outcome where it succeeds, one for each rejection it declares, and one for each policy that reacts to what it records. When the business describes a new rule, write its story first, watch it fail, then change the model until it passes.

## Where to go next

You’ve now met every building block in the manual. To put them to work together, build the bank in the [tutorial](tutorial/part-1.md). For every check a story offers, answering several external calls, and asserting real ones, see [Testing a model](testing-models.md). To test what clients see through an Api, including middleware, see [Testing an Api](testing-apis.md).
