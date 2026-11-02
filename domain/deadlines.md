---
url: https://alchemy.run/domain/deadlines
title: "Deadlines"
description: "Give a waiting process a time limit with a scheduled command, and let the aggregate decide whether the deadline or the action wins."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A saga that waits needs a deadline. A large transfer waits for a support agent’s approval, and expires if nobody approves it within 24 hours. Schedule a command for the deadline, and let the aggregate decide whichever comes first.

## Declare a command that reports the deadline

A scheduled command fires when nobody is waiting for its answer, so it never rejects. It reports a fact, and records nothing when the fact no longer applies.

```typescript
import * as Domain from "@alchemy.run/domain";
import { TransferFailed } from "./FailTransfer.command.ts";
import { TransferId } from "./Transfer.id.ts";

/**
 * The approval window closed. Scheduled by \`TransferApprovalDeadline\`; it
 * reports a fact, so it never rejects, and it records nothing if the transfer
 * was approved in time.
 */
export class ExpireTransfer extends Domain.Command("ExpireTransfer", {
  target: TransferId,
  events: [TransferFailed],
}) {}
```

## Schedule it

A policy schedules the command when the waiting starts.

```typescript
import * as Domain from "@alchemy.run/domain";
import * as Effect from "effect/Effect";
import { ExpireTransfer } from "./ExpireTransfer.command.ts";
import { TransferRequested } from "./RequestTransfer.command.ts";
import { Transfer } from "./Transfer.aggregate.ts";

/** How long a support agent has to approve a large transfer. */
export const APPROVAL_WINDOW = "24 hours";

/**
 * Whenever a transfer needs approval, expire it if nobody approves it in time.
 * The Transfer decides the race: approval and expiry are commands on the same
 * instance, so exactly one of them wins.
 */
export class TransferApprovalDeadline extends Domain.Policy("TransferApprovalDeadline", {
  from: Transfer,
  on: [TransferRequested],
}) {}

export const TransferApprovalDeadlineLive = TransferApprovalDeadline.toLayer(
  Effect.gen(function* () {
    const transfers = yield* Transfer;
    return Effect.fn(function* ({ source, event }) {
      if (!event.needsApproval) return;
      yield* transfers.schedule(source, new ExpireTransfer(), { after: APPROVAL_WINDOW }).pipe(
        // A redelivery after the window closed: the expiry already ran.
        Effect.catchTag("AlreadyExecuted", () => Effect.void),
      );
    });
  }),
);
```

The schedule is durable: it fires even if everything restarted in between. Scheduling again with the same command id replaces the earlier one.

A scheduled command takes its command id like any other command, so it runs once. Scheduling an id the transfer already handled fails with `AlreadyExecuted`, because that command would never run again. Here it means the policy ran a second time for the same event, after the expiry fired, so there’s nothing to do.

## Let the aggregate decide the race

Approval and expiry are both commands to the same transfer. One may come from a support agent at any moment, the other from the schedule after 24 hours.

<svg viewBox="0 0 700 352" role="img" aria-label="Both commands reach the same transfer, which handles one at a time."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 296 L 262 296" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 88 L 350 176 L 506 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 264 L 350 176 L 506 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">Support agent</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">approves</text></g></g> <g transform="translate(24, 264)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">Schedule</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">set by the policy</text></g> <g transform="translate(268, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">ApproveTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">any time</text></g></g> <g transform="translate(268, 264)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">ExpireTransfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">after 24 hours</text></g></g> <g transform="translate(512, 144)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Transfer</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one at a time</text></g></g></svg>

CommandAggregateClient

Both commands reach the same transfer, which handles one at a time.

The transfer handles one command at a time, so whichever arrives first changes its state, and its `decide` turns the other away. Approval is only accepted while the transfer is awaiting approval:

```typescript
ApproveTransfer: Effect.fn(function* (s, _, { id }) {
  const approvedBy = yield* CurrentActor;
  if (approvedBy._tag !== "Agent") return yield* new NotPermitted();
  if (s._tag !== "AwaitingApproval") return yield* new NotAwaitingApproval();
```

Expiry fails the transfer only if it’s still waiting, and records nothing otherwise:

```typescript
ExpireTransfer: (s, _, { id }) =>
  s._tag === "AwaitingApproval"
    ? [new TransferFailed({ transferId: id, reason: "not approved in time" })]
    : [],
```

So an approved transfer ignores its expiry, and an expired transfer rejects a late approval with `NotAwaitingApproval`.

## Cancel it

When the deadline no longer applies, cancel it. Schedule the command with a command id you can name again, and unschedule it by that id:

```typescript
// when the waiting starts
yield* transfers.schedule(source, new ExpireTransfer(), {
  after: APPROVAL_WINDOW,
  commandId: \`expire:${source.id}\`,
});

// when the transfer is approved
yield* transfers.unschedule(source, \`expire:${source.id}\`);
```

`schedule` returns a `ScheduleReceipt` with the command id and when the command is due (`at`), so code that schedules without its own id can still cancel by `receipt.commandId`.

Unscheduling is durable, like scheduling. Once it succeeds, the command never fires, even when a story’s clock passes its time. When there was nothing to cancel, it fails and says why: `AlreadyExecuted` when the command already ran, `AlreadyCancelled` when it was cancelled before, and `NotScheduled` when nothing was scheduled with that id.

A policy cancels after the approval is recorded, so the expiry can still arrive in between. Keep the guard in `decide` for that case. A policy can also run again for the same event, and then finds the expiry already cancelled:

```typescript
yield* transfers.unschedule(source, \`expire:${source.id}\`).pipe(
  Effect.catchTag(["AlreadyExecuted", "AlreadyCancelled"], () => Effect.void),
);
```

## Test it with the clock

A story has its own clock. `Story.clock` moves it, and any scheduled command that’s now due fires.

```typescript
// A minute before the deadline, nothing has happened.
Story.clock("2026-01-02T08:59:00Z"),
Story.then(transfer),
// At the deadline, the transfer expires.
Story.clock("2026-01-02T09:00:00Z"),
Story.then(transfer, new TransferFailed({ transferId: transfer.id, reason: "not approved in time" })),
// Approval now comes too late.
Story.provide(CurrentActor, agent("support-7")),
Story.when(transfer, new ApproveTransfer()),
Story.rejected(new NotAwaitingApproval()),
```

`Story.then(transfer)` with no events checks that nothing new was recorded.
