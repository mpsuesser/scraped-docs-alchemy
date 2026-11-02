---
url: https://alchemy.run/domain/saga
title: "Saga"
description: "A process that spans several aggregates and takes time, run step by step as an Effect workflow, with steps that undo earlier ones when a later one fails."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A saga runs a process that spans several aggregates. An aggregate changes one instance at a time, and no transaction spans two, so a process that touches several runs as a series of steps, each committed on its own. When a step fails after earlier ones succeeded, the saga undoes them with compensations.

A saga is an [Effect workflow](https://effect.website/docs/workflow). Each step’s result is stored, so a run that’s interrupted resumes after its last finished step, and a run can wait hours or days for a signal or a timer. A policy usually decides when to start one.

In the example bank, a transfer is a saga: it debits one account, credits another, and refunds the debit if the credit fails.

<svg viewBox="0 0 944 352" role="img" aria-label="A policy decides when the transfer runs; the saga carries it out, one step at a time."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 56 L 750 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 838 88 L 838 138" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 838 88 L 838 258" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">RequestTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">command</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Transfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">records the request</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(5.700000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="25.700000000000003" y="28" dominant-baseline="middle">TransferExecution</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">starts the saga</text></g></g> <g transform="translate(756, 24)"><g style="--hue:#c6d84a"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path fill="none" d="M3 3h7v7H3z"></path><path fill="none" d="M14 14h7v7h-7z"></path><path stroke="currentColor" fill="none" d="M6.5 10v4a3 3 0 0 0 3 3H14"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">ExecuteTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">runs the steps</text></g></g> <g transform="translate(756, 144)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">DebitTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">1 · source account</text></g></g> <g transform="translate(756, 264)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">2 · destination</text></g></g></svg>

CommandAggregatePolicySaga

A policy decides when the transfer runs; the saga carries it out, one step at a time.

This chapter starts with why a process across aggregates can’t be one transaction, and what to do instead. Then it builds the transfer saga step by step.

## No transaction spans two aggregates

Moving money between two accounts touches two aggregates, and each is its own consistency boundary. There’s no step in which both change together. So a transfer can’t be “debit and credit, or neither”, the way a database transaction would make it.

Instead, it’s a sequence of steps, each committed on its own: debit the source, then credit the destination. In between, the money is in flight. It has left one account and not yet reached the other, and anyone reading both balances at that moment sees it. That’s not a bug to hide; it’s how money moves in real banks too, and the `Transfer` aggregate records that the transfer is pending so screens can show it.

## Undo with a new fact, not an erasure

When a later step fails, the earlier ones have already happened. Their events are facts and can’t be erased. So the saga undoes them the way a business would, by recording another fact that reverses the effect: a debited transfer whose credit is refused is refunded, and the account’s history shows the debit and the refund, one after the other.

<svg viewBox="0 0 700 112" role="img" aria-label="A compensation doesn't rewrite history. It adds the step that puts things right."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">TransferDebited</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a-1 · −50</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a-2 is closed</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">TransferRefunded</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a-1 · +50</text></g></g></svg>

EventRejection

A compensation doesn't rewrite history. It adds the step that puts things right.

These undo steps are called **compensations**. Each step that can be undone declares its compensation next to it, and a saga that fails runs them in reverse order. A compensation must not be refused, or the process would be stuck halfway, so the command it sends records a fact in every state. See [Never refuse a compensation](aggregate.md#never-refuse-a-compensation).

## Saga or policy?

A [policy](policy.md) is one reaction: an event arrives, the policy acts, and it’s done. A saga is a process with memory. It knows which step it’s on, waits between steps, and knows what to undo if something fails.

Reach for a saga when a process has more than one step, waits for something, a decision or a timer, or has to undo earlier steps on failure. A policy usually starts the saga, on the event that records the request.

## Declare a saga

A saga has a key, the input it starts with, and every way it can be refused. The key identifies one run: starting the saga again with the same transfer id returns the same run.

```typescript
export class ExecuteTransfer extends Domain.Saga("ExecuteTransfer", {
  key: TransferId,
  input: {
    from: AccountId,
    to: AccountId,
    amount: Cents,
    requestedBy: Actor,
    needsApproval: Schema.Boolean,
  },
  rejects: [NotOpen, NotOwner, Frozen, InsufficientFunds, DestinationClosed],
}) {}
```

## Run its steps

Its Layer resolves the clients its steps use, then returns what each run does. Each step is an `Activity`: a named Effect whose result is stored. When the run replays, a finished step returns its stored result instead of running again.

```typescript
yield* Activity.make({
  name: "credit",
  error: DestinationClosed,
  execute: accounts
    .send(to, new CreditTransfer({ transferId: key, amount }))
    .pipe(Effect.catchTag("NotOpen", () => Effect.fail(new DestinationClosed()))),
});
```

A step declares the errors it can fail with, so a refusal survives being stored. Here the destination account’s `NotOpen` becomes the saga’s own `DestinationClosed`.

A step that runs again is safe. A step can run twice: when it’s retried, or when its run is cut short after the step’s command committed but before its result was stored. The commands a step sends get ids derived from the run and the step, so the second run sends the same command ids, and the aggregate applies each command once. Keep each step to one command, so each stored result matches one fact.

## Undo a step when a later one fails

A step can carry a compensation: what to do if the run fails after the step succeeded. The debit’s compensation refunds it.

```typescript
yield* Activity.make({
  name: "debit",
  error: Schema.Union([NotOpen, NotOwner, Frozen, InsufficientFunds]),
  execute: accounts.send(from, new DebitTransfer({ transferId: key, amount, requestedBy })),
}).pipe(
  ExecuteTransfer.withCompensation(() =>
    Activity.make({
      name: "refund",
      execute: accounts.send(from, new RefundTransfer({ transferId: key, amount })),
    }),
  ),
);
```

When the credit is refused, the run fails, and every compensation registered so far runs, last step first.

<svg viewBox="0 0 700 232" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 594 88 L 594 138" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">DebitTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">succeeded</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">refused</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(5.700000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="25.700000000000003" y="28" dominant-baseline="middle">DestinationClosed</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the saga fails</text></g></g> <g transform="translate(512, 144)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">RefundTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">undoes the debit</text></g></g></svg>

CommandRejection

## Take the step you can’t undo first

Debit before you credit. If the credit then fails, the money is refunded, so it can be in flight but never created out of nothing. In the same way, registration claims the username before the customer counts as registered: see [Unique values](unique-values.md).

## Record the outcome

The Transfer aggregate shows where every transfer stands, so the saga reports back to it. The last step completes the transfer:

```typescript
yield* Activity.make({
  name: "complete",
  execute: transfers.send(key, new CompleteTransfer()),
});
```

A failure is recorded by a finalizer. It’s registered first, so it runs after every compensation:

```typescript
yield* Workflow.addFinalizer((exit) =>
  Exit.isSuccess(exit)
    ? Effect.void
    : Activity.make({
        name: "record-failure",
        execute: transfers.send(key, new FailTransfer({ reason: reasonOf(exit.cause) })),
      }).pipe(Effect.orDie),
);
```

## Start it from a policy

Start a saga from a policy, on the event that records the request. The command that asked for the transfer commits `TransferRequested` and replies straight away; the policy starts the saga after that, and the saga runs on its own. Nobody waits for it.

<svg viewBox="0 0 700 232" role="img" aria-label="The request is a fact before the process starts, so it's never lost, and the caller never waits for the process."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 594 88 L 594 138" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 512 176 L 438 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">RequestTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">replies at once</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Transfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">records the request</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(5.700000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="25.700000000000003" y="28" dominant-baseline="middle">TransferRequested</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the fact</text></g></g> <g transform="translate(512, 144)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(5.700000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="25.700000000000003" y="28" dominant-baseline="middle">TransferExecution</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">starts it, later</text></g></g> <g transform="translate(268, 144)"><g style="--hue:#c6d84a"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path fill="none" d="M3 3h7v7H3z"></path><path fill="none" d="M14 14h7v7h-7z"></path><path stroke="currentColor" fill="none" d="M6.5 10v4a3 3 0 0 0 3 3H14"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">ExecuteTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">runs on its own</text></g></g></svg>

CommandAggregateEventPolicySaga

The request is a fact before the process starts, so it's never lost, and the caller never waits for the process.

`yield* ExecuteTransfer` gives the saga’s client. `start` begins a run, or returns the one already started with that key, as a handle to the run. Policies run at least once, so key the saga by the aggregate it’s about: a policy that runs twice still starts one transfer.

```typescript
case "TransferRequested":
  yield* executions.start(source, {
    from: event.from,
    to: event.to,
    amount: event.amount,
    requestedBy: event.requestedBy,
    needsApproval: event.needsApproval,
  });
  return;
```

List the saga on the model, next to its aggregates and policies, and its Layer with the other handlers.

```typescript
sagas: [ExecuteTransfer],
```

## Don’t wait for it in a policy

`start` returns as soon as the run is recorded. Its run handle also has `result`, which waits for the run to finish, but a run can take days, and a policy that waits holds up every later event on its key. Use `result` in tests and scripts. Anything that needs to react to how a saga ends should react to the events it reports instead, like `TransferCompleted` or `TransferFailed`.

## Wait for a decision

A run can wait as long as it needs to. A large transfer waits until the Transfer decides it: an agent approves it, or its approval window closes first. The saga waits on a `DurableDeferred`, and suspends until someone completes it.

```typescript
if (needsApproval) {
  // The Transfer decides between approval and expiry; nothing to do if it expired.
  if ((yield* DurableDeferred.await(Decision)) === "expired") return;
}
```

The policy passes on the decision with the run’s `signal`. `get` returns the run with a key:

```typescript
case "TransferApproved":
  return yield* executions.get(source).signal(Decision, "approved");
```

The Transfer decides the race between approval and expiry, so there’s one source of truth for it. See [Deadlines](deadlines.md).

## Follow it

`TransferStatus` is a view of the transfer and both accounts. A client watches it to see the saga finish: the saga reports every step to the Transfer, so the view has everything a client should see.

```typescript
export const followTransfer = Effect.fn(function* (bank: CustomerClient, transferId: string) {
  yield* bank.TransferStatus.watch(TransferId.make(transferId)).pipe(
    Stream.map(({ state }) => state),
    Stream.takeUntil((status) => status?.completedAt !== null || status?.failedReason !== null),
    Stream.runForEach((status) =>
      Effect.log(status === null ? "pending" : TransferStatus.computed.status(status)),
    ),
  );
});
```

The saga’s own client reads a run too, which suits operators and tests more than apps. `get` returns the run with a key:

```typescript
const run = executions.get(transferId);
const status = yield* run.status;
```

`status` is `NotStarted`, `Running`, `Completed` with its result, `Rejected` with its rejection, `Interrupted`, or `Died` with a defect. Runs read the same as [Effect workflow runs](../cloudflare/compute/effect-workflows.md), on every engine.

## Test it

A story runs the whole saga in memory: the request, every step, the compensations, and its status at the end.

Test what a saga does, not how it’s built: the commands its steps send show up as events on the aggregates, and `Story.saga` checks how the run ended, with the same status its run handle reads.

```typescript
Story.saga(ExecuteTransfer, transfer, { _tag: "Rejected", rejection: new DestinationClosed() }),
```

To test a saga on its own, complete what it waits for with `Story.signal`, and move its durable clocks with `Story.clock`:

```typescript
Story.saga(ExecuteTransfer, transfer, { _tag: "Running" }),
Story.signal(ExecuteTransfer, transfer, Decision, "approved"),
```

A step that calls an external system waits until the story answers the call, as a policy does. A run that dies fails the story, unless a step expects it with `Story.saga(ExecuteTransfer, transfer, { _tag: "Died" })`.

A saga runs wherever its model runs: in memory in a story, and on [Cloudflare Durable Objects](cloudflare.md) in production, with each run’s steps stored durably.

Next, [External System](external-system.md): how the model talks to the world outside it.
