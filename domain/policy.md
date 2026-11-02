---
url: https://alchemy.run/domain/policy
title: "Policy"
description: "How a policy reacts to events with commands and calls to outside systems, and how to make it safe to run more than once."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A policy is one of the business’s standing rules about what to do when something happens. In the example bank, whenever a customer registers, their main account is opened, and whenever an account’s withdrawals for the day cross a threshold, it’s scored for fraud and frozen if the risk is high.

The name comes from business policy, and that’s the point: a policy doesn’t just react, it decides. It’s triggered by an event, works out whether and how to act, and acts by sending commands to aggregates or calling systems outside the model.

That’s a different kind of rule from an aggregate’s. An aggregate enforces invariants, which must hold the instant a command commits and can refuse it. A policy applies a procedure, which runs after the event and decides what happens next. This chapter covers how to find the policies in a business, how to declare and implement one, and the one property every policy must have, which is that running it twice does no harm.

## Find the policies in your business

Policies hide in sentences that start with “whenever” or “when … then”. Whenever a customer registers, open their main account. When money settles on a closed account, refund it. When a large transfer is requested, wait for an agent to approve it. People in the business say these all the time, often as “we always …” or “the rule is that …”.

Each sentence has two halves. The “whenever” names an event, which is what the policy listens for. The “then” names what the business does next, which becomes a command to an aggregate or a call to a system outside the model. Between them sits whatever judgment the business applies: the fraud rule only freezes an account when the score is high.

<svg viewBox="0 0 700 112" role="img" aria-label="A policy is a sentence the business already says, written as code."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Whenever …</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">CustomerRegistered</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">the rule</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">open a main account</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">… then</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">OpenAccount</text></g></g></svg>

CommandEventPolicy

A policy is a sentence the business already says, written as code.

Writing these down as policies, instead of burying them in the code that handles a request, keeps them visible. The registration command only registers. The rule about main accounts has its own name, its own file and its own tests, and changing it touches nothing else.

## Policy or invariant?

Before you write a policy, check that the rule can tolerate a delay. A policy runs shortly after the event, so there’s always a moment when the event has happened and the policy hasn’t acted yet.

“Open a main account for every new customer” is fine with that: for a moment, a customer has no account. “A frozen account can’t be withdrawn from” isn’t. If a policy enforced it, a withdrawal could slip through between the freeze and the policy noticing. Rules that must hold at every instant are invariants, and they belong in an aggregate’s `decide`. Rules about what happens next are policies.

## Declare what triggers it

A policy names its source, an aggregate or a view, and the events from it that it reacts to.

```typescript
export class MainAccountProvisioning extends Domain.Policy("MainAccountProvisioning", {
  from: Customer,
  on: [CustomerRegistered],
}) {}
```

The declaration is all the model needs to deliver the right events. Add it to the model’s `policies`.

`from` can also list several sources. Each source’s events still arrive in that source’s order, but there is no order between them: see [Partition it by key](#partition-it-by-key).

## Implement it as a Layer

The implementation is separate from the declaration, because it usually depends on things outside the model’s definition: a client for another aggregate, a payment provider, a fraud service. `toLayer` takes an Effect that resolves those once, and returns the handler that runs for each event.

```typescript
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

The handler receives the event and its `source`, the instance that recorded it. Here the source is the customer who registered, and the policy opens an account whose id is derived from theirs.

You pass the implementation to the model when you run it: `Bank.layer(BankHandlers)`. See [Domain Model](model.md#declare-the-model).

<svg viewBox="0 0 700 112" role="img" aria-label="The registration commits first. The policy runs shortly after and opens the account."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">Customer c-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">aggregate</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">policy</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">on CustomerRegistered</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">Account c-1-main</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">OpenAccount</text></g></g></svg>

AggregatePolicy

The registration commits first. The policy runs shortly after and opens the account.

## It runs after the commit

A policy never runs in the same step as the event it reacts to. The customer’s registration commits on its own; then the event is delivered to the policy, which sends a separate command to a separate aggregate.

So there’s a moment when the customer exists and their account doesn’t yet. That’s fine for opening an account, and it’s the only way to coordinate two aggregates, since each is its own consistency boundary. It’s not fine for a rule that must hold the instant a command commits, like “a frozen account can’t be withdrawn from”. That rule belongs in the aggregate’s `decide`, even when it’s about the aggregate’s own events. See [Aggregate](aggregate.md#one-command-at-a-time).

<svg viewBox="0 0 700 112" role="img" aria-label="Two commits, in two aggregates. Between them, the customer exists and the account doesn't yet."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">commit 1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Customer c-1</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">policy</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">MainAccountProvisioning</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">commit 2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Account c-1-main</text></g></g></svg>

AggregatePolicy

Two commits, in two aggregates. Between them, the customer exists and the account doesn't yet.

## Make it safe to run twice

Every event reaches every policy that reacts to it, in order for each source instance, even across restarts. To guarantee that, the model may deliver an event more than once: if a handler fails or the host restarts midway, it’s run again.

Two things make a second run harmless. First, every command a policy sends gets an id derived from the event that caused it. A second run sends the same command with the same id, and the aggregate returns the receipt it stored the first time instead of deciding again. See [Command](command.md#send-it-exactly-once).

Second, a policy handles the rejections it expects from a repeat. Opening an account that’s already open is refused with `AlreadyOpen`, so the policy treats that as done:

```typescript
.pipe(Effect.catchTag("AlreadyOpen", () => Effect.void));
```

Calls to outside systems need the same care. Pass them an idempotency key derived from the event, so the payment provider charges once however many times it’s asked.

## Partition it by key

By default, a policy is partitioned by source instance: everything one customer records is handled in order, one event at a time, and different customers run in parallel.

When a rule spans several sources, choose the partition yourself with `key`, and route each event to it with `keyOf`, exactly as a view does. Here one welcome flow per customer hears both the registration and every account the customer opens, in one ordered queue.

```typescript
export class Welcome extends Domain.Policy("Welcome", {
  from: [Customer, Account],
  key: Customer,
  keyOf: {
    AccountOpened: ({ event }) => Customer.ref(CustomerId.make(event.owner)),
  },
  on: [CustomerRegistered, AccountOpened],
}) {}
```

`CustomerRegistered` comes from a customer, so it needs no route. `AccountOpened` comes from an account, so `keyOf` names the customer it belongs to. The handler receives that customer as `key`.

<svg viewBox="0 0 456 204" role="img" aria-label="Two sources, one partition per customer. Within it, events from different sources have no order between them."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">Customer c-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">CustomerRegistered</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">Account c-1-main</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">AccountOpened</text></g></g> <g transform="translate(268, 70)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Welcome c-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one ordered queue</text></g></g></svg>

AggregatePolicy

Two sources, one partition per customer. Within it, events from different sources have no order between them.

`key` can also be a string Schema, such as an org id. `keyOf` then returns the value, and the handler gets it as `key`.

```typescript
export class OrgLimits extends Domain.Policy("OrgLimits", {
  from: Account,
  key: OrgId,
  keyOf: { MoneyWithdrawn: ({ event }) => event.orgId },
  on: [MoneyWithdrawn],
}) {}
```

The key is also the policy’s unit of concurrency. Pick the narrowest thing whose events must be handled in sequence: everything for one key runs one at a time, and more keys mean more parallelism.

## Tune it on its Layer

What a policy means lives on its declaration. How it runs lives on its Layer. `toLayer` takes options, such as a retry schedule for a run that fails before the platform’s own redelivery takes over.

```typescript
export const WelcomeLive = Welcome.toLayer(
  Effect.succeed(
    Effect.fn(function* ({ key, event }) {
      yield* Effect.log(\`welcome ${key.id}: ${event._tag}\`);
    }),
  ),
  { retry: Schedule.exponential("1 second").pipe(Schedule.upTo({ duration: "1 minute" })) },
);
```

## React to a view

A policy can listen to a view as well as an aggregate. That lets it react to patterns no single event shows.

The example bank wants to review an account when its withdrawals for one day add up past a threshold, even if each one is small. No withdrawal event says that. So a view, `DailyOutflow`, keeps a running total per account per day, and emits `DailyOutflowExceeded` when the total crosses the line. The policy listens for that.

```typescript
export class FraudReview extends Domain.Policy("FraudReview", {
  from: DailyOutflow,
  on: [DailyOutflowExceeded],
}) {}
```

<svg viewBox="0 0 944 204" role="img" aria-label="DailyOutflow emits DailyOutflowExceeded; FraudReview scores the account and freezes it if it's risky."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 102 C 225 102, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 102 C 469 102, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 102 C 713 102, 713 56, 750 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 102 C 713 102, 713 148, 750 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 70)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">aggregate</text></g></g> <g transform="translate(268, 70)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">DailyOutflow a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">view</text></g></g> <g transform="translate(512, 70)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">FraudReview</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">policy</text></g></g> <g transform="translate(756, 24)"><g style="--hue:#d97aa8"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M9 2v6"></path><path stroke="currentColor" fill="none" d="M15 2v6"></path><path fill="none" d="M6 8h12v4a6 6 0 0 1-12 0z"></path><path stroke="currentColor" fill="none" d="M12 18v4"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">FraudCheck</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">external system</text></g></g> <g transform="translate(756, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">FreezeAccount</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">command</text></g></g></svg>

CommandAggregateViewPolicyExternal system

DailyOutflow emits DailyOutflowExceeded; FraudReview scores the account and freezes it if it's risky.

## Call an external system

To decide whether the account is risky, the policy asks a fraud-scoring service. It doesn’t call a vendor’s SDK directly. It depends on a `Domain.ExternalSystem`, which names the outside system and the calls the model makes to it.

```typescript
export class FraudCheck extends Domain.ExternalSystem("FraudCheck", {
  score: { args: { accountId: AccountId, amount: Cents }, success: { risk: Schema.Number } },
}) {}
```

The policy resolves it like any other service, then calls it per event.

```typescript
const fraud = yield* FraudCheck;
const accounts = yield* Account;
return Effect.fn(function* ({ source, event }) {
  const { risk } = yield* fraud.score({ accountId: source.id, amount: event.total });
```

An external system is implemented by a Layer, so production can use the vendor’s HTTP API while a test uses something else. In a [story](story.md), each call waits until the test answers it, so you can check what the policy does with every possible answer. See [External System](external-system.md).

## Act as the system

Commands sent by people pass through middleware that says who they are. A policy has no caller, so when a command’s `decide` needs to know who’s acting, the policy says so itself.

```typescript
yield* accounts.send(source, new FreezeAccount({ reason: \`fraud risk ${risk}\` })).pipe(
  Effect.provideService(CurrentActor, system("FraudReview")),
```

The freeze is then recorded as done by `FraudReview`, so the account’s history says why it was frozen and by what.

## Chain policies into a process

One policy covers one reaction, and it should stay small: hear one kind of event, decide one thing, send one command or make one call. When you find a policy waiting for a second event, tracking how far along something is, or undoing what it did earlier, it has grown into a process. A process with several steps, like a transfer that debits one account, credits another, and refunds the first if the second fails, is a [saga](saga.md). The policy stays small: it hears that a transfer was requested and starts the saga.

<svg viewBox="0 0 944 112" role="img" aria-label="One process, three steps across two accounts. If the credit is refused, a refund undoes the debit."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 56 C 713 56, 713 56, 750 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">Transfer t-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">TransferRequested</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">DebitTransfer</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">CreditTransfer</text></g></g> <g transform="translate(756, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">RefundTransfer</text></g></g></svg>

Aggregate

One process, three steps across two accounts. If the credit is refused, a refund undoes the debit.

Next, [Saga](saga.md): a process that takes several steps, and undoes them when one fails.
