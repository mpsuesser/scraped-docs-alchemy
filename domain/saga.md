---
url: https://alchemy.run/domain/saga
title: "Saga"
description: "A process that spans several aggregates and takes time, run step by step as an Effect workflow, with steps that undo earlier ones when a later one fails."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A saga runs a process that spans several aggregates. An aggregate changes one instance at a time, and no transaction spans two, so a process that touches several runs as a series of steps, each committed on its own. When a step fails after earlier ones succeeded, the saga undoes them with compensations.

A saga is an [Effect workflow](https://effect.website/docs/workflow). Each step’s result is stored, so a run that’s interrupted resumes after its last finished step, and a run can wait hours or days for a signal or a timer. A policy usually decides when to start one.

In the example bank, a transfer is a saga: it debits one account, credits another, and refunds the debit if the credit fails.

<svg viewBox="0 0 1012 352" style="max-width: 1164px" role="img" aria-label="A policy decides when the transfer runs; the saga carries it out, one step at a time."><defs><marker id="alc-dag-arrow-f3tm28" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 205 56 L 279 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-f3tm28)"></path></g><g><path d="M 466 56 L 540 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-f3tm28)"></path></g><g><path d="M 727 56 L 801 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-f3tm28)"></path></g><g><path d="M 897.5 88 L 897.5 138" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-f3tm28)"></path></g><g><path d="M 897.5 88 L 897.5 258" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-f3tm28)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 169 L 181 32 L 169 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(22, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="42" y="28" dominant-baseline="middle">RequestTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">command</text></g></g> <g transform="translate(285, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="181" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="173" height="56" rx="5"></rect><g transform="translate(49.3, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="69.3" y="28" dominant-baseline="middle">Transfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">records the request</text></g></g> <g transform="translate(546, 24)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 172 L 181 9 V 55 L 172 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(14.200000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="34.2" y="28" dominant-baseline="middle">TransferExecution</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">starts the saga</text></g></g> <g transform="translate(807, 24)"><g style="--hue:#c6d84a"><rect stroke="currentColor" fill="none" width="181" height="64" rx="8"></rect><g transform="translate(22, 21) scale(0.5833333333333334)"><path fill="none" d="M3 3h7v7H3z"></path><path fill="none" d="M14 14h7v7h-7z"></path><path stroke="currentColor" fill="none" d="M6.5 10v4a3 3 0 0 0 3 3H14"></path></g><text fill="currentColor" x="42" y="28" dominant-baseline="middle">ExecuteTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">runs the steps</text></g></g> <g transform="translate(807, 144)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 169 L 181 32 L 169 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(29.800000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="49.800000000000004" y="28" dominant-baseline="middle">DebitTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">1 · source account</text></g></g> <g transform="translate(807, 264)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 169 L 181 32 L 169 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.900000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.900000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">2 · destination</text></g></g></svg> <svg viewBox="0 0 394 580" style="max-width: 453px" role="img" aria-label="A policy decides when the transfer runs; the saga carries it out, one step at a time."><defs><marker id="alc-dag-arrow-d98vqm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 197 74 C 197 101, 197 101, 197 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-d98vqm)"></path></g><g><path d="M 197 198 C 197 225, 197 225, 197 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-d98vqm)"></path></g><g><path d="M 197 322 C 197 349, 197 349, 197 376" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-d98vqm)"></path></g><g><path d="M 197 446 C 197 473, 96.5 473, 96.5 500" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-d98vqm)"></path></g><g><path d="M 197 446 C 197 473, 297.5 473, 297.5 500" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-d98vqm)"></path></g><g transform="translate(110.5, 10)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 161 L 173 32 L 161 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(18, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="38" y="28" dominant-baseline="middle">RequestTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">command</text></g></g> <g transform="translate(110.5, 134)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="173" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="165" height="56" rx="5"></rect><g transform="translate(45.3, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="65.3" y="28" dominant-baseline="middle">Transfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">records the request</text></g></g> <g transform="translate(110.5, 258)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 164 L 173 9 V 55 L 164 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(10.200000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="30.200000000000003" y="28" dominant-baseline="middle">TransferExecution</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">starts the saga</text></g></g> <g transform="translate(110.5, 382)"><g style="--hue:#c6d84a"><rect stroke="currentColor" fill="none" width="173" height="64" rx="8"></rect><g transform="translate(18, 21) scale(0.5833333333333334)"><path fill="none" d="M3 3h7v7H3z"></path><path fill="none" d="M14 14h7v7h-7z"></path><path stroke="currentColor" fill="none" d="M6.5 10v4a3 3 0 0 0 3 3H14"></path></g><text fill="currentColor" x="38" y="28" dominant-baseline="middle">ExecuteTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">runs the steps</text></g></g> <g transform="translate(10, 506)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 161 L 173 32 L 161 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.800000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.800000000000004" y="28" dominant-baseline="middle">DebitTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">1 · source account</text></g></g> <g transform="translate(211, 506)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 161 L 173 32 L 161 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.900000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.900000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">2 · destination</text></g></g></svg>

CommandAggregatePolicySaga

A policy decides when the transfer runs; the saga carries it out, one step at a time.

## No transaction spans two aggregates

Moving money between two accounts touches two aggregates, and each is its own consistency boundary. There’s no step in which both change together. So a transfer can’t be “debit and credit, or neither”, the way a database transaction would make it.

Instead, it’s a sequence of steps, each committed on its own: debit the source, then credit the destination. In between, the money is in flight. It has left one account and not yet reached the other, and anyone reading both balances at that moment sees it. That’s not a bug to hide; it’s how money moves in real banks too, and the `Transfer` aggregate records that the transfer is pending so screens can show it.

## Undo with a new fact, not an erasure

When a later step fails, the earlier ones have already happened. Their events are facts and can’t be erased. So the saga undoes them the way a business would, by recording another fact that reverses the effect: a debited transfer whose credit is refused is refunded, and the account’s history shows the debit and the refund, one after the other.

<svg viewBox="0 0 727 112" style="max-width: 836px" role="img" aria-label="A compensation doesn't rewrite history. It adds the step that puts things right."><defs><marker id="alc-dag-arrow-b30vkm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 197 56 L 271 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-b30vkm)"></path></g><g><path d="M 450 56 L 524 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-b30vkm)"></path></g><g transform="translate(24, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 160 L 173 13 V 60 Q 173 64 169 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 160 0 V 13 H 173"></path><g transform="translate(18, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="38" y="28" dominant-baseline="middle">TransferDebited</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">a-1 · −50</text></g></g> <g transform="translate(277, 24)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="173" height="64" rx="8"></rect><g transform="translate(21.900000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="41.900000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">a-2 is closed</text></g></g> <g transform="translate(530, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 160 L 173 13 V 60 Q 173 64 169 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 160 0 V 13 H 173"></path><g transform="translate(14.099999999999994, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="34.099999999999994" y="28" dominant-baseline="middle">TransferRefunded</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">a-1 · +50</text></g></g></svg> <svg viewBox="0 0 185 324" style="max-width: 213px" role="img" aria-label="A compensation doesn't rewrite history. It adds the step that puts things right."><defs><marker id="alc-dag-arrow-34slha" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 92.5 74 L 92.5 124" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-34slha)"></path></g><g><path d="M 92.5 194 L 92.5 244" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-34slha)"></path></g><g transform="translate(10, 10)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 152 L 165 13 V 60 Q 165 64 161 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 152 0 V 13 H 165"></path><g transform="translate(14, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="34" y="28" dominant-baseline="middle">TransferDebited</text> <text fill="currentColor" x="82.5" y="46" text-anchor="middle" dominant-baseline="middle">a-1 · −50</text></g></g> <g transform="translate(10, 130)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="165" height="64" rx="8"></rect><g transform="translate(17.900000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="37.900000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="82.5" y="46" text-anchor="middle" dominant-baseline="middle">a-2 is closed</text></g></g> <g transform="translate(10, 250)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 152 L 165 13 V 60 Q 165 64 161 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 152 0 V 13 H 165"></path><g transform="translate(10.099999999999994, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="30.099999999999994" y="28" dominant-baseline="middle">TransferRefunded</text> <text fill="currentColor" x="82.5" y="46" text-anchor="middle" dominant-baseline="middle">a-1 · +50</text></g></g></svg>

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

## Describe it

A saga’s last argument is its [annotations](annotations.md). Its `description` is for the people and the tools that read the model, as `ExecuteTransfer.annotations`.

```typescript
export class ExecuteTransfer extends Domain.Saga(
  "ExecuteTransfer",
  {
    key: TransferId,
    input: {
      from: AccountId,
      to: AccountId,
      amount: Cents,
      requestedBy: Actor,
      needsApproval: Schema.Boolean,
    },
    rejects: [NotOpen, NotOwner, Frozen, InsufficientFunds, DestinationClosed],
  },
  { description: "Debits one account, credits another, and refunds the debit if the credit fails." },
) {}
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

<svg viewBox="0 0 751 232" style="max-width: 864px" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow-nump2m" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 205 56 L 279 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-nump2m)"></path></g><g><path d="M 466 56 L 540 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-nump2m)"></path></g><g><path d="M 636.5 88 L 636.5 138" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-nump2m)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 169 L 181 32 L 169 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(29.800000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="49.800000000000004" y="28" dominant-baseline="middle">DebitTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">succeeded</text></g></g> <g transform="translate(285, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 169 L 181 32 L 169 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.900000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.900000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">refused</text></g></g> <g transform="translate(546, 24)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="181" height="64" rx="8"></rect><g transform="translate(14.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="34.2" y="28" dominant-baseline="middle">DestinationClosed</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">the saga fails</text></g></g> <g transform="translate(546, 144)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 169 L 181 32 L 169 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.900000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.900000000000006" y="28" dominant-baseline="middle">RefundTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">undoes the debit</text></g></g></svg> <svg viewBox="0 0 394 324" style="max-width: 453px" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow-p269qs" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 96.5 74 L 96.5 124" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-p269qs)"></path></g><g><path d="M 96.5 194 L 96.5 244" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-p269qs)"></path></g><g><path d="M 183 282 L 205 282" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-p269qs)"></path></g><g transform="translate(10, 10)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 161 L 173 32 L 161 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.800000000000004, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.800000000000004" y="28" dominant-baseline="middle">DebitTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">succeeded</text></g></g> <g transform="translate(10, 130)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 161 L 173 32 L 161 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.900000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.900000000000006" y="28" dominant-baseline="middle">CreditTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">refused</text></g></g> <g transform="translate(10, 250)"><g style="--hue:#e5484d"><rect stroke="currentColor" fill="none" width="173" height="64" rx="8"></rect><g transform="translate(10.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M9 9l6 6"></path><path stroke="currentColor" fill="none" d="M15 9l-6 6"></path></g><text fill="currentColor" x="30.200000000000003" y="28" dominant-baseline="middle">DestinationClosed</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">the saga fails</text></g></g> <g transform="translate(211, 250)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 161 L 173 32 L 161 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(21.900000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="41.900000000000006" y="28" dominant-baseline="middle">RefundTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">undoes the debit</text></g></g></svg>

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

<svg viewBox="0 0 751 232" style="max-width: 864px" role="img" aria-label="The request is a fact before the process starts, so it's never lost, and the caller never waits for the process."><defs><marker id="alc-dag-arrow-xviszs" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 205 56 L 279 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-xviszs)"></path></g><g><path d="M 466 56 L 540 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-xviszs)"></path></g><g><path d="M 636.5 88 L 636.5 138" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-xviszs)"></path></g><g><path d="M 546 176 L 472 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-xviszs)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 169 L 181 32 L 169 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(22, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="42" y="28" dominant-baseline="middle">RequestTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">replies at once</text></g></g> <g transform="translate(285, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="181" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="173" height="56" rx="5"></rect><g transform="translate(49.3, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="69.3" y="28" dominant-baseline="middle">Transfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">records the request</text></g></g> <g transform="translate(546, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 168 L 181 13 V 60 Q 181 64 177 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 168 0 V 13 H 181"></path><g transform="translate(14.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="34.2" y="28" dominant-baseline="middle">TransferRequested</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">the fact</text></g></g> <g transform="translate(546, 144)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 172 L 181 9 V 55 L 172 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(14.200000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="34.2" y="28" dominant-baseline="middle">TransferExecution</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">starts it, later</text></g></g> <g transform="translate(285, 144)"><g style="--hue:#c6d84a"><rect stroke="currentColor" fill="none" width="181" height="64" rx="8"></rect><g transform="translate(22, 21) scale(0.5833333333333334)"><path fill="none" d="M3 3h7v7H3z"></path><path fill="none" d="M14 14h7v7h-7z"></path><path stroke="currentColor" fill="none" d="M6.5 10v4a3 3 0 0 0 3 3H14"></path></g><text fill="currentColor" x="42" y="28" dominant-baseline="middle">ExecuteTransfer</text> <text fill="currentColor" x="90.5" y="46" text-anchor="middle" dominant-baseline="middle">runs on its own</text></g></g></svg> <svg viewBox="0 0 394 324" style="max-width: 453px" role="img" aria-label="The request is a fact before the process starts, so it's never lost, and the caller never waits for the process."><defs><marker id="alc-dag-arrow-7phfao" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 96.5 74 L 96.5 124" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-7phfao)"></path></g><g><path d="M 96.5 194 L 96.5 244" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-7phfao)"></path></g><g><path d="M 183 282 L 205 282" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-7phfao)"></path></g><g><path d="M 297.5 250 L 297.5 200" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-7phfao)"></path></g><g transform="translate(10, 10)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 161 L 173 32 L 161 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(18, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="38" y="28" dominant-baseline="middle">RequestTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">replies at once</text></g></g> <g transform="translate(10, 130)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="173" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="165" height="56" rx="5"></rect><g transform="translate(45.3, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="65.3" y="28" dominant-baseline="middle">Transfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">records the request</text></g></g> <g transform="translate(10, 250)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 160 L 173 13 V 60 Q 173 64 169 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 160 0 V 13 H 173"></path><g transform="translate(10.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="30.200000000000003" y="28" dominant-baseline="middle">TransferRequested</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">the fact</text></g></g> <g transform="translate(211, 250)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 164 L 173 9 V 55 L 164 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(10.200000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="30.200000000000003" y="28" dominant-baseline="middle">TransferExecution</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">starts it, later</text></g></g> <g transform="translate(211, 130)"><g style="--hue:#c6d84a"><rect stroke="currentColor" fill="none" width="173" height="64" rx="8"></rect><g transform="translate(18, 21) scale(0.5833333333333334)"><path fill="none" d="M3 3h7v7H3z"></path><path fill="none" d="M14 14h7v7h-7z"></path><path stroke="currentColor" fill="none" d="M6.5 10v4a3 3 0 0 0 3 3H14"></path></g><text fill="currentColor" x="38" y="28" dominant-baseline="middle">ExecuteTransfer</text> <text fill="currentColor" x="86.5" y="46" text-anchor="middle" dominant-baseline="middle">runs on its own</text></g></g></svg>

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

A saga runs wherever its model runs: in memory in a story, and on [Cloudflare](cloudflare.md), [Rivet](rivet.md) or [Effect Cluster](effect-cluster.md) in production, with each run’s steps stored durably.

Next, [External System](external-system.md): how the model talks to the world outside it.
