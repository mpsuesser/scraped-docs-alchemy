---
url: https://alchemy.run/domain/what-is-ddd
title: "What is Domain-Driven Design?"
description: "The concepts of Domain-Driven Design, event sourcing and CQRS, and how they fit together to model a business."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

Domain-Driven Design (DDD) is an approach to building software in which the code is a model of the business it serves. Instead of starting from tables and endpoints, you start from the business itself: what it deals with, what can happen, who decides, and what people need to know. The code is then organized around those answers.

This page introduces the concepts DDD uses to do that, along with two techniques that are often paired with it, event sourcing and CQRS. The examples come from a bank, the same one the [tutorial](tutorial/part-1.md) builds.

## The domain and its model

The **domain** is the area of activity your software serves. For a bank, that’s customers, accounts, deposits, withdrawals, transfers and fraud. The domain exists whether or not there’s any software; it’s the business as the people in it understand it.

A **domain model** is a deliberate simplification of the domain, written as code. It keeps what matters for the decisions the software has to make and leaves the rest out. A bank’s model needs to know an account’s balance, because withdrawals depend on it. It doesn’t need to know the colour of the customer’s card.

The model is about behaviour, not storage. It says what can happen in the domain, which rules govern it, and what’s remembered as a result. How that’s stored is a separate concern.

Here is the shape of the bank’s model, which the rest of this page explains piece by piece. A customer’s app sends commands; aggregates decide them and record events; views turn events into what the app reads; and policies turn events into further commands.

<svg viewBox="0 0 1000 260" role="img" aria-label="Clients send commands and read views. Dashed edges happen shortly after the events are recorded."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 102 C 225 102, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 102 C 469 102, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 102 C 713 102, 713 56, 750 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 102 C 713 102, 713 148, 750 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 838 180 C 838 216, 106 216, 106 140" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 70)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Commands</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">DepositMoney</text></g></g> <g transform="translate(268, 70)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Aggregates</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Account</text></g></g> <g transform="translate(512, 70)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Events</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">MoneyDeposited</text></g></g> <g transform="translate(756, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(52.5, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="72.5" y="28" dominant-baseline="middle">Views</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">AccountSummary</text></g></g> <g transform="translate(756, 116)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Policies</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">FraudReview</text></g></g></svg>

CommandAggregateEventViewPolicy

Clients send commands and read views. Dashed edges happen shortly after the events are recorded.

## A shared language

DDD asks that the model and the people who run the business use the same words for the same things. If the bank says customers *deposit money* into an *account*, the code says `DepositMoney` and `Account`, not `updateBalance` and `rows`.

```typescript
export class AccountOpened extends Domain.Event("AccountOpened") {}
export class MoneyDeposited extends Domain.Event("MoneyDeposited") {}
export class InsufficientFunds extends Domain.Rejection("InsufficientFunds") {}
```

This shared vocabulary is called the **ubiquitous language**. It means a conversation about a rule turns directly into a change in the code, and reading the code tells you how the business works. When the business has a word for something, the model uses it. When the model needs a word the business lacks, that’s a sign something hasn’t been understood yet.

## Commands: what someone wants to happen

A **command** is a request to change something in the domain: open an account, deposit 100, withdraw 30. It’s named as an instruction, in the imperative.

A command is only a request. It can be refused: a withdrawal from an account that doesn’t hold enough money is turned down. So a command also declares the ways it can be refused.

```typescript
export class WithdrawMoney extends Domain.Command("WithdrawMoney", {
  target: AccountId,
  input: { amount: Schema.Int },
  events: [MoneyWithdrawn],
  rejects: [NotOpen, InsufficientFunds],
  reply: { balance: Schema.Int },
}) {}
```

See [Command](command.md).

## Events: what happened

An **event** records something that has happened in the domain: an account was opened, money was deposited, money was withdrawn. It’s named in the past tense, because it’s a fact.

Facts can’t be refused or undone. If a deposit was a mistake, the bank doesn’t erase it; it records a correction as a new event, and the history shows both. So where a command may or may not succeed, an event already has.

```typescript
export class MoneyWithdrawn extends Domain.Event("MoneyWithdrawn", {
  amount: Schema.Int,
  balanceAfter: Schema.Int,
}) {}
```

Commands and events are the two halves of every change. Someone asks with a command; if the request is accepted, events record what happened.

See [Event](event.md).

## Aggregates: where decisions are made

Something has to decide whether a command is accepted. In DDD, that’s an **aggregate**: a cluster of related state together with the rules that govern it. A bank account is an aggregate. It holds a balance, and it enforces the rules about that balance, such as “a withdrawal can’t exceed the balance”.

Rules like that are called **invariants**: conditions that must be true at every moment, with no exceptions. An aggregate exists to protect its invariants. Every command that could affect them goes through the aggregate, which checks it against its current state and either refuses it or records events.

<svg viewBox="0 0 700 204" role="img" aria-label="An aggregate turns a command into events, or refuses it."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 102 C 225 102, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 102 C 469 102, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 102 C 469 102, 469 148, 506 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 70)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">WithdrawMoney</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">command</text></g></g> <g transform="translate(268, 70)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">aggregate</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">MoneyWithdrawn</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">event</text></g></g> <g transform="translate(512, 116)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(5.700000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="25.700000000000003" y="28" dominant-baseline="middle">InsufficientFunds</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">rejection</text></g></g></svg>

CommandAggregateEventRejection

An aggregate turns a command into events, or refuses it.

Each aggregate is also a **consistency boundary**. It handles one command at a time, so two withdrawals arriving together are decided one after the other, and the second sees the balance the first left behind. Everything inside the boundary is always consistent. Nothing outside it changes in the same step.

That makes the size of an aggregate a design decision. It should contain exactly the state its invariants need. A customer and their accounts are separate aggregates, because no rule requires a customer’s details and an account’s balance to change together.

Here is the whole account: how it decides each command, and how each event changes its state.

```typescript
export class Account extends Domain.Aggregate("Account", {
  id: AccountId,
  state: State,
  initial: { _tag: "Unopened" },
  commands: [OpenAccount, DepositMoney, WithdrawMoney],
  decide: {
    OpenAccount: (s, cmd) =>
      s._tag !== "Unopened" ? new AlreadyOpen() : [new AccountOpened({ owner: cmd.owner })],
    DepositMoney: (s, cmd) =>
      s._tag !== "Open"
        ? new NotOpen()
        : [new MoneyDeposited({ amount: cmd.amount, balanceAfter: s.balance + cmd.amount })],
    WithdrawMoney: (s, cmd) =>
      s._tag !== "Open"
        ? new NotOpen()
        : cmd.amount > s.balance
          ? new InsufficientFunds({ balance: s.balance, requested: cmd.amount })
          : [new MoneyWithdrawn({ amount: cmd.amount, balanceAfter: s.balance - cmd.amount })],
  },
  evolve: {
    AccountOpened: (_, e) => ({ _tag: "Open" as const, owner: e.owner, balance: 0 }),
    MoneyDeposited: (s, e) => (s._tag === "Open" ? { ...s, balance: e.balanceAfter } : s),
    MoneyWithdrawn: (s, e) => (s._tag === "Open" ? { ...s, balance: e.balanceAfter } : s),
  },
  reply: {
    DepositMoney: (s) => ({ balance: s._tag === "Open" ? s.balance : 0 }),
    WithdrawMoney: (s) => ({ balance: s._tag === "Open" ? s.balance : 0 }),
  },
}) {}
```

See [Aggregate](aggregate.md).

## Event sourcing: the events are the record

Most systems store the current state and overwrite it on every change. **Event sourcing** stores the events instead, and treats the current state as something derived from them.

An account’s balance is then the result of applying its events in order: opened, deposited 100, withdrew 30, so the balance is 70. Nothing about the account is lost. You can always answer why the balance is what it is, rebuild it from scratch, or derive a new kind of information from history you already have.

This is why an aggregate has two parts. **Decide** looks at the current state and a command, and returns events or a refusal. **Evolve** applies one event to the state to produce the next state. Decide may refuse; evolve never does, because the event it applies has already happened. Applying the same events always produces the same state.

<svg viewBox="0 0 1188 204" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 102 C 469 102, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 102 C 713 102, 713 102, 750 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 920 102 C 957 102, 957 102, 994 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">balance: 100</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">state</text></g> <g transform="translate(24, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">WithdrawMoney 30</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">command</text></g></g> <g transform="translate(268, 74)"><rect stroke="currentColor" fill="none" width="164" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="29" text-anchor="middle" dominant-baseline="middle">decide</text></g> <g transform="translate(512, 70)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">MoneyWithdrawn</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">event</text></g></g> <g transform="translate(756, 74)"><rect stroke="currentColor" fill="none" width="164" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="29" text-anchor="middle" dominant-baseline="middle">evolve</text></g> <g transform="translate(1000, 70)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">balance: 70</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">state</text></g></svg>

CommandEvent

## CQRS: deciding and answering are separate

An account needs its balance to decide a withdrawal. It doesn’t need a statement, a list of this month’s transactions, or a total across every account a customer holds. Those are questions people ask, and each needs the data in a different shape.

**CQRS**, Command Query Responsibility Segregation, splits the system along that line. The **command side** handles changes: commands go to aggregates, which decide and record events. The **query side** answers questions: it builds read-only models from those same events, each shaped for the question it answers. The two sides share nothing but the events.

<svg viewBox="0 0 700 232" role="img" aria-label="The command side records events. The query side builds views from them, and reads go to the views."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 594 88 L 594 176 L 438 176" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 268 176 L 194 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Commands</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">change something</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Aggregates</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">decide</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Events</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the record</text></g></g> <g transform="translate(268, 144)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(52.5, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="72.5" y="28" dominant-baseline="middle">Views</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one per question</text></g></g> <g transform="translate(24, 144)"><g style="--hue:#e6d3a8"><path fill="none" d="M 12 0 H 158 Q 164 0 164 6 V 58 Q 164 64 158 64 H 12 L 0 32 Z"></path><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M11 4a7 7 0 1 0 0 14a7 7 0 1 0 0-14z"></path><path stroke="currentColor" fill="none" d="M20 20l-4-4"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Queries</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">read the views</text></g></g></svg>

CommandAggregateEventViewQuery

The command side records events. The query side builds views from them, and reads go to the views.

The split lets each side do one job well. Aggregates stay small, holding only what their rules need. Views can be as numerous and as specialized as the screens that use them, and adding one never touches a rule. A new question about past activity can be answered by building a new view from the existing events.

## Views: models for reading

A **view**, also called a read model or projection, is the query side’s unit. It reads events, folds them into state, and keeps that state current as new events arrive. The account summary below keeps an owner and a balance for every account.

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

A client reads a view in one of two ways: fetch its current state once, or watch it for the state now and after every change.

```typescript
const summary = yield* bank.AccountSummary.get(accountId); // once
const changes = bank.AccountSummary.watch(accountId); // a stream of every change
```

Views are updated after events are recorded, not in the same step. A view is always correct, but it may be a moment behind the aggregate. That delay is called **eventual consistency**. A screen that has just sent a command and wants to show its effect waits until the view has caught up with it.

Not every read model belongs in the model’s own storage. A search index, a reporting database or a data warehouse is built the same way, by consuming events, but lives in a store you choose. That’s a **projection**: a read model the model feeds and you own. The bank’s ledger copies every money movement into a database table:

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

See [View](view.md) and [Projection](projection.md).

## Policies: the business’s standing rules

Businesses have rules about what to do when something happens. Whenever a customer registers, open their main account. Whenever an account’s withdrawals for a day pass a threshold, have it checked for fraud, and freeze it if the risk is high. These are business policies, and DDD models each one as a **policy**.

A policy is triggered by an event, decides whether and how to act, and acts by sending commands or calling systems outside the model. The fraud policy, for example, asks a scoring service and only freezes the account if the score is high.

<svg viewBox="0 0 700 112" role="img" aria-label="The registration is recorded first. The policy then sends OpenAccount to a different aggregate."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Customer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">CustomerRegistered</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">policy</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">open main account</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">OpenAccount</text></g></g></svg>

AggregatePolicy

The registration is recorded first. The policy then sends OpenAccount to a different aggregate.

The main-account policy is declared with what triggers it, and implemented as a Layer that sends the command:

```typescript
export class MainAccountProvisioning extends Domain.Policy("MainAccountProvisioning", {
  from: Customer,
  on: [CustomerRegistered],
}) {}

export const MainAccountProvisioningLive = MainAccountProvisioning.toLayer(
  Effect.gen(function* () {
    const accounts = yield* Account;
    return Effect.fn(function* ({ source }) {
      yield* accounts
        .send(AccountId.make(\`${source.id}-main\`), new OpenAccount({ owner: source.id }))
        .pipe(Effect.catchTag("AlreadyOpen", () => Effect.void));
    });
  }),
);
```

Policies and aggregates both apply rules, but different kinds. An aggregate enforces invariants: rules that must hold the instant a command commits, inside one boundary, and that can refuse the command. A policy applies a procedure: what the business does next, often across aggregates or outside the model. It runs after the event, and it can’t refuse what already happened. It can only decide what happens next, through commands that the receiving aggregates can still refuse.

When a process takes several steps across aggregates and can fail partway, like a transfer that debits one account and then credits another, a **saga** drives it step by step, and undoes earlier steps if a later one fails. A policy usually starts it. See [Saga](saga.md).

## Testing: given, when, then

Designing this way makes the model easy to test, because every piece is defined by events. An aggregate’s behaviour is a function of its history and a command. A view’s state is a function of the events it has read. A policy’s actions are a function of the event that triggered it.

So a test needs no database, no mocks and no setup code. It states the history as events, sends one command, and checks the outcome as events, or as a rejection:

<svg viewBox="0 0 700 232" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 88 L 350 176 L 506 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(52.5, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="72.5" y="28" dominant-baseline="middle">given</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">events so far</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(56.4, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="76.4" y="28" dominant-baseline="middle">when</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a command</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(56.4, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="76.4" y="28" dominant-baseline="middle">then</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">new events</text></g></g> <g transform="translate(512, 144)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">or rejected</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a rejection</text></g></g></svg>

CommandEventRejection

That’s also how the business states its rules: “an account holding 100 refuses a withdrawal of 150”. The test reads the same way:

```typescript
it.effect("refuses to withdraw more than the balance", () =>
  story(
    Story.given(
      account,
      new AccountOpened({ owner: "sam" }),
      new MoneyDeposited({ amount: 100, balanceAfter: 100 }),
    ),
    Story.when(account, new WithdrawMoney({ amount: 150 })),
    Story.rejected(new InsufficientFunds({ balance: 100, requested: 150 })),
  ),
);
```

The same shape covers the rest of the model. A view is tested by giving it events and checking its state. A policy is tested by checking the commands it sends, and calls to external systems wait for the test to answer them, so every outcome of an outside call can be tried. Because the whole model runs in memory, a test of a process spanning several aggregates still runs in milliseconds.

See [Story](story.md).

## Bounded contexts: one language per model

Large businesses don’t have a single language. To the bank, an “account” holds money. To the support team’s ticketing system, an “account” is a login. One model that tries to serve both ends up serving neither.

DDD splits the business into **bounded contexts**: areas in which one model and one language apply consistently. Each has its own aggregates, views and policies. Contexts talk to each other through published interfaces, and never by reaching into each other’s aggregates.

<svg viewBox="0 0 756 168" role="img" aria-label="Two bounded contexts, each with its own meaning of Account, talking through an Api."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 594 88 C 594 130, 350 130, 350 94" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 88 C 350 130, 106 130, 106 94" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Banking: holds money</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#ece8df"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M3 12h18"></path><path stroke="currentColor" fill="none" d="M12 3c3 3 3 15 0 18"></path><path stroke="currentColor" fill="none" d="M12 3c-3 3-3 15 0 18"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">BankApi</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">published interface</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Identity: a login</text></g></g></svg>

AggregateApi

Two bounded contexts, each with its own meaning of Account, talking through an Api.

## How @alchemy.run/domain models these concepts

Each concept on this page is a building block in `@alchemy.run/domain`. A bounded context is a [`Domain.Model`](model.md), and its published interface is an [`Api`](api.md). Inside it, you declare [commands](command.md), [events](event.md), [aggregates](aggregate.md) with `decide` and `evolve`, [views](view.md) and [policies](policy.md). The platform you run the model on provides the guarantees described here: one command at a time per aggregate, events and state committed together, and every event delivered to every view and policy that consumes it, in order.

A model lists its building blocks:

```typescript
export class Bank extends Domain.Model("Bank", {
  aggregates: [Account, Customer],
  views: [AccountSummary, Statement],
  policies: [MainAccountProvisioning],
}) {}
```

Next, build one: the [tutorial](tutorial/part-1.md) puts these ideas to work on a small bank, one step at a time. Or read the concepts in depth as a manual, one chapter per building block, starting with [Domain Model](model.md).
