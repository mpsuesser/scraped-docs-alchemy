---
url: https://alchemy.run/domain/model
title: "Domain Model"
description: "What a domain model contains, how it processes every command, and how you declare, run and use one."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A `Domain.Model` is one bounded context running as a whole. It holds the context’s aggregates, the views built from their events, and the policies that act on them, and it guarantees how commands and events move between them.

The concept pages that follow read as a manual, one building block per chapter, in the order a command meets them. This first chapter is the map. It describes what every model contains and what happens to every command, with the bank from the [tutorial](tutorial/part-1.md) as the example throughout, then shows how to declare, run and call a model.

If the words aggregate, view and policy are new, read [What is Domain-Driven Design?](what-is-ddd.md) first.

## A model is a bounded context

A business is too big for one model. The words its people use change meaning from one department to the next: to the bank’s tellers, an “account” holds money; to its identity team, an “account” is a login. A single model that tries to serve both meanings ends up with a vague `Account` that serves neither.

So DDD draws boundaries. Inside a **bounded context**, every word means exactly one thing, and the model is written in those words. In the example bank, the banking context knows accounts, deposits, withdrawals and transfers. It doesn’t know about login sessions, password resets or support tickets; those belong to other contexts, with models of their own.

A good boundary passes two tests. Every name inside it has one meaning, so a conversation with the business turns straight into code. And the rules inside it can be enforced without asking another context’s permission, so the model can decide on its own. When a word starts needing a qualifier, like “billing account” versus “login account”, you’ve found the edge of a context.

A `Domain.Model` is how you write one bounded context down. Everything in this manual lives inside one.

## What a model contains

Every model has the same shape. Commands enter on the left and are decided by aggregates. Aggregates record events. Events flow to three kinds of consumer: views, which the model stores and serves; projections, which write a store you own or a stream; and policies, which send further commands.

<svg viewBox="0 0 1000 352" role="img" aria-label="Solid edges happen within one command. Dashed edges are delivered after it commits."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 148, 506 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 148 C 713 148, 713 56, 750 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 148 C 713 148, 713 148, 750 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 676 148 C 713 148, 713 240, 750 240" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 838 272 C 838 308, 106 308, 106 186" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Commands</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">requests</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Aggregates</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">decide</text></g></g> <g transform="translate(512, 116)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Events</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the record</text></g></g> <g transform="translate(756, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(52.5, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="72.5" y="28" dominant-baseline="middle">Views</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">answer reads</text></g></g> <g transform="translate(756, 116)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Projections</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">write stores</text></g></g> <g transform="translate(756, 208)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Policies</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">act next</text></g></g></svg>

CommandAggregateEventViewProjectionPolicy

Solid edges happen within one command. Dashed edges are delivered after it commits.

A model declares which aggregates, views, policies, projections and streams it contains. The commands and events come with them, since each aggregate declares the commands it handles and the events it records. The example bank’s model contains two aggregates, two views and one policy:

```typescript
export class Bank extends Domain.Model("Bank", {
  aggregates: [Account, Customer],
  views: [AccountSummary, Statement],
  policies: [MainAccountProvisioning],
}) {}
```

## Each command goes to one instance

An aggregate type has many instances, one per id, and every command names the id of the instance it’s for. The model routes the command to that instance and no other.

Each instance handles one command at a time. Commands for the same instance queue, and each is decided against the state the previous one left. Commands for different instances run independently. This is what makes an instance a consistency boundary: nothing else changes its state while it decides.

<svg viewBox="0 0 456 296" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 102, 262 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 240 C 225 240, 225 194, 262 194" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">command</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">for id 1</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">command</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">for id 1</text></g></g> <g transform="translate(24, 208)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">command</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">for id 2</text></g></g> <g transform="translate(268, 70)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">instance 1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one at a time</text></g></g> <g transform="translate(268, 162)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">instance 2</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">one at a time</text></g></g></svg>

CommandAggregate

In the bank, a deposit names the account it’s for, so two deposits into `a-1` queue while a deposit into `a-2` runs alongside them:

```typescript
const receipt = yield* accounts.send(accountId, new DepositMoney({ amount: 100 }));
```

## The aggregate decides, and everything commits at once

The instance runs the aggregate’s `decide` handler for the command against its current state. A rejection ends the command: nothing is recorded, and the caller gets the rejection as a typed error. Otherwise, `decide` returns events, and each is applied to the state with `evolve`.

Then the instance commits, in one atomic write: the events, appended to its stream with sequence numbers; its new state and version; the reply; and a delivery for every view and policy that consumes the events. Either all of it is stored or none of it is, so a crash can never leave events without their deliveries, or state that disagrees with its events.

<svg viewBox="0 0 700 388" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 194 C 225 194, 225 194, 262 194" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 194 C 469 194, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 194 C 469 194, 469 148, 506 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 194 C 469 194, 469 240, 506 240" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 194 C 469 194, 469 332, 506 332" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 162)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">decide</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">accepted</text></g> <g transform="translate(268, 166)"><rect stroke="currentColor" fill="none" width="164" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="29" text-anchor="middle" dominant-baseline="middle">one atomic write</text></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">events</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">appended</text></g></g> <g transform="translate(512, 116)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">state</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">new version</text></g> <g transform="translate(512, 208)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">receipt</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">with the reply</text></g> <g transform="translate(512, 300)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">deliveries</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">views, policies</text></g></svg>

Event

The caller gets a receipt with the stream and version the command committed at, the events, and the reply. The receipt is also stored under the command’s id. If the same command arrives again, after a retry over a flaky connection, the instance returns the stored receipt instead of deciding twice.

For a deposit into an open account, `decide` returns one `MoneyDeposited` event carrying the new balance:

```typescript
DepositMoney: (s, cmd) =>
  s._tag !== "Open"
    ? new NotOpen()
    : [new MoneyDeposited({ amount: cmd.amount, balanceAfter: s.balance + cmd.amount })],
```

## Consumers catch up

After the commit, the deliveries are sent. Every view, projection and policy receives every event it consumes, at least once, and in the order its source instance recorded them, even if something restarts in between. Views and projections drop events they’ve already accepted; policies make repeats harmless (see [Policy](policy.md#make-it-safe-to-run-twice)).

Because this happens after the commit, views are eventually consistent: for a moment, the aggregate has the new state and a view doesn’t yet. A reader who needs to see their own change passes the command’s receipt, and the view waits until it has applied that version.

In the bank, one `MoneyDeposited` reaches both views that read accounts:

<svg viewBox="0 0 456 204" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 102 C 225 102, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 102 C 225 102, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 70)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">MoneyDeposited</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Account a-1</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">balance</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(36.9, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="56.9" y="28" dominant-baseline="middle">Statement</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a new line</text></g></g></svg>

EventView

And a client that has just deposited reads the summary at that receipt, so it sees the new balance:

```typescript
return yield* summaries.query(Account.ref(accountId), { atLeast: receipt });
```

## Consumers added later receive the history

Add a view or a projection to a model that already has events, and it receives those events first. You deploy, and it builds itself from the history, with no command and no script to run.

Each key of the new consumer pulls the history from every instance that feeds it and applies those events in the order they were recorded, by timestamp. A summary per account sees its account’s events in order, and a single dashboard fed by every account sees all of them in order, as if it had been there from the start. Events recorded while it catches up wait their turn, then arrive live.

Policies never receive history. They act on new events only, so adding a policy never acts on the past again.

## Change a view: bump its version

A view’s state is derived, so it can always be computed again. Whenever you change what a view computes (its `evolve`, `emit`, `keyOf`, `state`, or the events and sources it reads), bump its `version`:

```typescript
import * as Domain from "@alchemy.run/domain";
import * as Schema from "effect/Schema";
import { Account } from "./Account.aggregate.ts";

export class AccountSummary extends Domain.View("AccountSummary", {
  from: [Account],
  key: Account,
  state: Schema.Struct({ net: Schema.Number }),
  initial: { net: 0 },
  evolve: {
    MoneyDeposited: (v, e) => ({ net: v.net + e.amount }),
    MoneyWithdrawn: (v, e) => ({ net: v.net - e.amount }),
  },
  version: 1,
}) {}
```

On the next deploy every key is computed again from the history, with the new code, and so is every view and projection that reads this one. You don’t choose how: the deploy picks the cheapest way that is correct.

A view keyed by the one aggregate it reads, like this summary per account, is rebuilt in place. It keeps its storage, and the deploy waits for nothing. The first time a key is touched after the deploy, it reads its own account’s events again before it answers, so even events it never received before (here, withdrawals) count. A view or projection that reads it gets each key’s new rows, never a key still being rebuilt. On Cloudflare, an object busy during the deploy keeps its old code until it is idle; until then the Worker computes that key’s answers itself, from its account’s events.

Any other view, such as one fed by many accounts through `keyOf`, is rebuilt from the history: it starts over in new storage and receives every event again, and the deploy waits for it, the same way it does for a new view.

Only ever increase a version. To undo a change, change the code back and bump again. Bump it too when you add back a view you removed earlier: its stored state stopped following the history while it was gone.

## Fill a view lazily

A view keyed by the one aggregate it reads can skip the backfill when it is added, too. List it as lazy on the model’s Layer:

```typescript
import * as Layer from "effect/Layer";
import { Bank } from "./Bank.model.ts";

export const BankLive = Bank.layer(Layer.empty, { lazy: ["AccountSummary"] });
```

A deploy that adds it waits for nothing. Each key reads its own account’s history the first time it’s read, or when that account next changes, and every answer it gives covers the whole history. Keys nobody reads are never built.

A view fed by many instances, like a total across every account, can’t be lazy: one key doesn’t know which instances feed it. The model fails to build if you try.

## Know when the history has arrived

Until a new or rebuilt view has received the history, its answers cover only part of it. `history` says whether it’s complete:

```typescript
const { complete } = yield* summaries.history;
```

A screen can show “catching up” until `complete` is true, or hold its reads until then. A projection’s `Progress` service has the same `history`.

On Cloudflare the deploy itself waits until the history is complete. See [History at scale](backfill.md) for how long that takes and what it costs.

## Order and partitions

Every consumer works the same way underneath. At commit, each event is routed to a key of each consumer that wants it: by default its source instance, or whatever the consumer’s `key` and `keyOf` say. Each key has an ordered input log, and its inputs are handled one at a time, or a batch at a time for a projection. Keys run in parallel.

That gives a precise guarantee. Within a key, events from one source arrive in that source’s order. Between keys, and between different sources feeding one key, there is no order: an account can be a day ahead of the customer that owns it. The model is distributed on purpose, so design consumers to correlate by id, use the envelope’s `at` for business time, and let an aggregate decide whenever the order between two facts matters.

## Choose a consumer

All three consumers receive the same events. They differ in what they do with them.

A [view](view.md) folds events into state that the model stores and serves: `get`, `watch`, `events`, with `validate` and read-your-own-write. Reach for it first when clients need to read something.

A [projection](projection.md) writes events into a store you own, such as a database or a search index, or into a [stream](stream.md) the model serves. It never sends commands, so it can always be replayed.

A [policy](policy.md) is a business rule. It acts on each new event once, through commands and external systems, and never replays.

## Declare the model

A model lists what runs together. It can start as a single aggregate.

```typescript
import * as Domain from "@alchemy.run/domain";
import { Account } from "./Account.aggregate.ts";

export class Bank extends Domain.Model("Bank", {
  aggregates: [Account],
}) {}
```

Adding a view to `views` is enough for the model to deliver the events it reads. You never wire an event to a view by hand.

```typescript
import * as Domain from "@alchemy.run/domain";
import { Account } from "./Account.aggregate.ts";
import { AccountSummary } from "./AccountSummary.view.ts";

export class Bank extends Domain.Model("Bank", {
  aggregates: [Account],
  views: [AccountSummary],
}) {}
```

A policy is declared in one place and implemented in another, because its implementation usually needs things from outside the model, like a client for another aggregate or a payment provider. The declaration goes in `policies`; the implementation is a Layer you keep beside it.

```typescript
import * as Domain from "@alchemy.run/domain";
import { Account } from "./Account.aggregate.ts";
import { AccountSummary } from "./AccountSummary.view.ts";
import { Customer } from "./Customer.aggregate.ts";
import {
  MainAccountProvisioning,
  MainAccountProvisioningLive,
} from "./MainAccountProvisioning.policy.ts";

export class Bank extends Domain.Model("Bank", {
  aggregates: [Account, Customer],
  views: [AccountSummary],
  policies: [MainAccountProvisioning],
}) {}

export const BankHandlers = MainAccountProvisioningLive;
```

## Check the declaration

Some mistakes in a declaration only show when the model is hosted: a view that reads a source keyed by another aggregate without `keyOf`, a view with `state` but no `initial`, a policy that waits for an event no source emits, a stream with two writers. Building the model’s Layer fails on the first one. `Domain.verify` finds all of them without hosting anything, so one unit test covers the whole model:

```typescript
import * as Domain from "@alchemy.run/domain";
import { expect, it } from "@effect/vitest";
import { Bank } from "../src/Bank.model.ts";

it("the model is well defined", () => {
  expect(Domain.verify(Bank)).toEqual([]);
});
```

Each problem is a `Domain.DefinitionError`. Its `reason` is a tagged error that says which problem it is, such as `MissingKeyOf` or `TwoWriters`, with the names involved as fields, and its message names the block and the fix. Mistakes that depend on what a handler returns, like `decide` returning an event its command doesn’t declare, still fail when the handler runs.

## Run it

A declaration doesn’t run anything. `Bank.layer(BankHandlers)` turns it into a Layer that hosts the model, and a platform underneath decides where its state lives.

```typescript
import * as Domain from "@alchemy.run/domain";
import * as Layer from "effect/Layer";
import { Bank, BankHandlers } from "./Bank.model.ts";

export const BankInMemory = Bank.layer(BankHandlers).pipe(Layer.provide(Domain.InMemory));
```

`Domain.InMemory` keeps everything in memory, which is what tests want. Swap it for another platform and the model runs there unchanged; every platform keeps the guarantees above. See [Hosting](cloudflare.md).

## Call it from your code

Inside the hosted model, each aggregate and view is a service. `yield* Account` gives you a client for every account, and `send` takes the target id and the command.

```typescript
import * as Effect from "effect/Effect";
import { Account } from "./Account.aggregate.ts";
import { AccountId } from "./Account.id.ts";
import { AccountSummary } from "./AccountSummary.view.ts";
import { DepositMoney } from "./DepositMoney.command.ts";

export const depositAndRead = Effect.gen(function* () {
  const accounts = yield* Account;
  const summaries = yield* AccountSummary;
  const accountId = AccountId.make("a-1");
  const receipt = yield* accounts.send(accountId, new DepositMoney({ amount: 100 }));
  return yield* summaries.query(Account.ref(accountId), { atLeast: receipt });
});
```

Passing the receipt as `atLeast` is the read-your-own-write from earlier: `query` waits until the summary has applied the deposit.

Clients outside the model, a browser or another service, reach it through an [Api](api.md) instead, which decides what each kind of client may send and read.

## Keep one language per model

A model is one bounded context, so every name in it means one thing. When two parts of the business use a word differently, give each its own model. One model never sends commands to another model’s aggregates directly; it calls that model’s Api.

That rule keeps each model free to change. A support tool built on another model’s Api depends on the commands and reads that model chose to publish, not on how its aggregates store their state today. The model can split an aggregate, rename a state or add a view without the support tool noticing.

<svg viewBox="0 0 700 112" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">Support</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Domain.Model</text></g> <g transform="translate(268, 24)"><g style="--hue:#ece8df"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M3 12h18"></path><path stroke="currentColor" fill="none" d="M12 3c3 3 3 15 0 18"></path><path stroke="currentColor" fill="none" d="M12 3c-3 3-3 15 0 18"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">BankApi</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Domain.Api</text></g></g> <g transform="translate(512, 24)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">Bank</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Domain.Model</text></g></svg>

Api

## Give each model its own data

Each model owns its data. Two models can use the same aggregate definition, and each still gets its own instances:

```typescript
class Billing extends Domain.Model("Billing", { aggregates: [Account], views: [Ledger] }) {}
class Audit extends Domain.Model("Audit", { aggregates: [Account], views: [Owners] }) {}
```

Account `a-1` in Billing and account `a-1` in Audit are different instances, with their own events. `Ledger` only hears Billing’s accounts, and `Owners` only hears Audit’s. Every platform keeps them apart, even on the same Worker; see [Cloudflare: Host several models in one Worker](cloudflare.md#host-several-models-in-one-worker). Give every model its own name: hosting two models with the same name on one platform fails.

## Read the rest of the manual

The chapters that follow take each building block in turn, in the order a command meets them.

A [Command](command.md) asks for a change, and an [Event](event.md) records one. An [Aggregate](aggregate.md) sits between them, deciding each command against the rules it protects. On the query side, a [View](view.md) answers a question from events, a [Projection](projection.md) feeds a store you own, and a [Stream](stream.md) keeps an ordered log a client can follow. A [Policy](policy.md) applies the business’s standing rules about what to do next, and a [Saga](saga.md) carries out a process that takes several steps. An [External System](external-system.md) is how the model talks to the world outside it, and an [Api](api.md) is what the model publishes to its clients. Finally, a [Story](story.md) states each rule as an example you can run.

Next, [Command](command.md): how a change is asked for.
