---
url: https://alchemy.run/rivet/concurrency
title: "Concurrency"
description: "How a Rivet Durable Object runs calls in parallel, how a transaction keeps a read and a write together, and how Workers.InputGate gives code written for Cloudflare the same rule it has there."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A room runs in one place, but it takes its calls in parallel. Two calls that reach the lobby at once both run, and their storage reads and writes interleave.

That’s fast: a busy room never waits for one call to finish before starting the next. It also means a method that reads a value and writes it back can lose another call’s write.

## Two joins, one lost

This `join` reads the visitor count, adds one, and writes it back:

```typescript
join: () =>
  Effect.gen(function* () {
    const visitors = ((yield* state.storage.get<number>("visitors")) ?? 0) + 1;
    yield* state.storage.put("visitors", visitors);
    return visitors;
  }),
```

When two joins reach a lobby with 3 visitors at once, both read 3, and both write 4:

<svg viewBox="0 0 640.4 120" style="max-width: 736px" role="img" aria-label="Both calls read before either writes, so the second write repeats the first and one visitor is lost."><g><text fill="currentColor" x="4" y="34">join A</text> <g><rect stroke="currentColor" fill="none" x="68.4" y="18" width="140" height="24" rx="5"></rect><text fill="currentColor" x="138.4" y="34" text-anchor="middle">get → 3</text></g> <g><rect stroke="currentColor" fill="none" x="278.4" y="18" width="140" height="24" rx="5"></rect><text fill="currentColor" x="348.4" y="34" text-anchor="middle">put 4</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="640.4" y2="52"></line><text fill="currentColor" x="4" y="78">join B</text> <g><rect stroke="currentColor" fill="none" x="138.4" y="62" width="139.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="208.39999999999998" y="78" text-anchor="middle">get → 3</text></g> <g><rect stroke="currentColor" fill="none" x="348.4" y="62" width="140" height="24" rx="5"></rect><text fill="currentColor" x="418.4" y="78" text-anchor="middle">put 4</text></g></g> <line stroke-opacity="0.2" stroke="currentColor" x1="68.4" y1="96" x2="628.4" y2="96"></line><text fill="currentColor" x="68.4" y="112" text-anchor="start">visitors = 3</text> <text fill="currentColor" x="628.4" y="112" text-anchor="end">visitors = 4</text></svg> <svg viewBox="0 0 330 140" style="max-width: 379px" role="img" aria-label="Both calls read before either writes, so the second write repeats the first and one visitor is lost."><g><text fill="currentColor" x="0" y="22">join A</text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="39.75" y="48" text-anchor="middle">get → 3</text></g> <g><rect stroke="currentColor" fill="none" x="119.25" y="32" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="159" y="48" text-anchor="middle">put 4</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">join B</text> <g><rect stroke="currentColor" fill="none" x="39.75" y="86" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="79.5" y="102" text-anchor="middle">get → 3</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="86" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="198.75" y="102" text-anchor="middle">put 4</text></g></g> <line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="318" y2="116"></line><text fill="currentColor" x="0" y="132" text-anchor="start">visitors = 3</text> <text fill="currentColor" x="318" y="132" text-anchor="end">visitors = 4</text></svg>

Both calls read before either writes, so the second write repeats the first and one visitor is lost.

## Keep a read and its write together

Run the change as a transaction. No other call’s storage interleaves with it, so the read and the write stay together:

```typescript
join: () =>
  state.storage.transaction(
    Effect.gen(function* () {
      const visitors = ((yield* state.storage.get<number>("visitors")) ?? 0) + 1;
      yield* state.storage.put("visitors", visitors);
      return visitors;
    }),
  ),
```

Fifty joins at once now count fifty visitors. The other calls’ storage waits for the transaction, while the rest of their work keeps running. A transaction also commits all its writes together and rolls them all back if it fails; see [Storage](storage.md#group-changes-in-a-transaction).

A change that is one write, like setting a key to a value that doesn’t depend on what was there, needs no transaction.

## Run Cloudflare code unchanged

Cloudflare’s Durable Objects follow a different rule. An **input gate** holds other calls back while one call’s storage runs, so a plain read-then-write never loses an update there. Code written for that rule, without transactions, loses updates on Rivet.

Provide `Workers.InputGate` to the class, and its instances follow Cloudflare’s rule:

```typescript
import * as Rivet from "@alchemy.run/rivet";
import * as Workers from "alchemy/Workers";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { Default } from "./cluster.ts";

export default class Counter extends Rivet.DurableObject<Counter>()(
  "Counter",
  { main: import.meta.url },
  Effect.gen(function* () {
    return Effect.gen(function* () {
      const state = yield* Rivet.DurableObjectState;
      return {
        increment: () =>
          Effect.gen(function* () {
            const count = ((yield* state.storage.get<number>("count")) ?? 0) + 1;
            yield* state.storage.put("count", count);
            return count;
          }),
      };
    });
  }).pipe(Effect.provide(Layer.mergeAll(Default, Workers.InputGate))),
) {}
```

Once a call starts using storage, no other call touches it until the first one waits on something else, like another Durable Object or an HTTP request. The `alarm` handler, work handed to `waitUntil`, and fibers the instance forks while it starts follow the same rule.

The gate costs throughput: every storage operation in the instance waits its turn. Give it to code written for Cloudflare, and use transactions everywhere else.

## Calls that would wait on themselves

A transaction holds the instance’s storage until it finishes. A call back into the same instance from inside one, directly or through other Durable Objects, would wait for the transaction while the transaction waits for it. Rivet spots that and fails the call back with HTTP 508, and the transaction sees the failure and rolls back. The same goes for an instance that calls itself while it starts.

Next, [Calling a Durable Object](calling.md): how callers reach a room.
