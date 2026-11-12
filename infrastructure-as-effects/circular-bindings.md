---
url: https://alchemy.run/infrastructure-as-effects/circular-bindings
title: "Circular Bindings"
description: "How to model two services that reference each other (Worker A ↔ Worker B, Lambda ↔ Lambda) using tagged classes and Layers."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Real systems have cycles. A web Worker calls an internal Worker for auth; the internal Worker calls back into the web Worker for billing. Two Lambdas trigger each other through a queue.

Most IaC engines reject these — you can’t create A before B if A needs B’s URL, but B needs A’s URL too. Alchemy resolves the deadlock by separating **identity** (a class used as a Tag) from **implementation** (a Layer attached via `.make()`).

Take two Workers that call each other, built one step at a time on [Bindings](binding.md) and [Layers](layers.md).

## Sketch the cycle

The goal: two Workers, `A` and `B`. Each one delegates half of its work to the other.

```plaintext
GET /  →  A  ──rpc──▶  B
              ◀──rpc───
```

Naively you’d write `import { B } from "./B.ts"` inside `A.ts` and `import { A } from "./A.ts"` inside `B.ts`. The imports succeed at the module level, but at deploy time neither Worker can be created first — each needs the other’s URL.

The fix is to import only the **Tag** (cheap, side-effect free) across the cycle, and keep each Worker’s **runtime implementation** behind a `.make()` call that runs only when the Stack provides it.

## Create A as a Tag

Start `src/A.ts` with just the class — no runtime yet. The class extends `Cloudflare.Worker<A, Shape>()(...)`, which produces a typed identifier you can `yield*` from anywhere. The second type parameter declares the shape A’s runtime will expose — here, a `work` method.

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";

export class A extends Cloudflare.Worker<A, { work: () => Effect.Effect<string> }>()("A") {}
```

`A` is now a Tag. It carries the type of A’s interface, but no implementation — importing it from another file does **not** force any runtime code to load, because there is no implementation yet. This is what makes the cycle resolvable.

## Create B as a Tag

Do the same for `B`. Both files only declare identity and shape at this point — no runtime.

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";

export class B extends Cloudflare.Worker<B, { work: () => Effect.Effect<string> }>()("B") {}
```

Now `A` and `B` can freely `import` each other — neither side triggers any runtime code by importing the other’s class.

## Add A’s runtime, binding B

Attach A’s implementation with `A.make(...)`. Inside the Construction phase, `yield* Cloudflare.Workers.bindWorker(B)` produces a typed stub whose methods dispatch to B’s deployed Worker at runtime. A also exposes its own `work` RPC method so B can call back into it.

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { B } from "./B.ts";

export class A extends Cloudflare.Worker<A, { work: () => Effect.Effect<string> }>()("A") {}

export default A.make(
  { main: import.meta.url },
  Effect.gen(function* () {
    const b = yield* Cloudflare.Workers.bindWorker(B);
    return {
      fetch: Effect.gen(function* () {
        // delegate half the work to B over the RPC stub
        return HttpServerResponse.text(yield* b.work());
      }),
      work: () => Effect.succeed("A handled its half"),
    };
  }),
);
```

The `import { B }` line pulls in B’s Tag only. `bindWorker(B)` returns a [typed RPC stub](../apis/schemaless.md) — at plan time it registers the service binding; at runtime `b.work()` round-trips to B’s deployed Worker as a native RPC call.

## Add B’s runtime, binding A

Mirror the same pattern in `B.ts`. B’s runtime imports A’s Tag and binds it.

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { A } from "./A.ts";

export class B extends Cloudflare.Worker<B, { work: () => Effect.Effect<string> }>()("B") {}

export default B.make(
  { main: import.meta.url },
  Effect.gen(function* () {
    const a = yield* Cloudflare.Workers.bindWorker(A);
    return {
      fetch: Effect.gen(function* () {
        return HttpServerResponse.text(yield* a.work());
      }),
      work: () => Effect.succeed("B handled its half"),
    };
  }),
);
```

This is the symmetric half of the cycle. `A` imports `B` ’s Tag and binds it; `B` imports `A` ’s Tag and binds it. Neither file needs the other’s `.make()` to be evaluated.

## Wire both into the Stack

The Stack pulls in both Tags (via `import { A }` / `import { B }`) and both Layers (via the default exports). `Effect.provide` attaches the Layers to the generator so the Tags resolve to real Workers.

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import ALive, { A } from "./src/A.ts";
import BLive, { B } from "./src/B.ts";

export default Alchemy.Stack(
  "AB",
  { providers: Cloudflare.providers(), state: Cloudflare.state() },
  Effect.gen(function* () {
    const a = yield* A;
    const b = yield* B;
    return { aUrl: a.url, bUrl: b.url };
  }).pipe(Effect.provide(Layer.mergeAll(ALive, BLive))),
);
```

`yield* A` and `yield* B` only succeed because `ALive` and `BLive` are provided. Forget one, and TypeScript flags the missing layer at the call site.

## How alchemy resolves the cycle

Under the hood, alchemy plans the cycle in two passes:

<svg viewBox="0 0 500 365" style="max-width: 575px" role="img" aria-label="Neither Worker waits for the other: each is reserved and created first, then a second pass wires in the real URLs."><defs><marker id="alc-seq-arrow-p49zoe" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-p49zoe" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="361"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="36" text-anchor="middle">alchemy deploy</text> <text fill="currentColor" x="82" y="51" text-anchor="middle">the plan</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="361"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">Worker A</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">binds B</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="361"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="36" text-anchor="middle">Worker B</text> <text fill="currentColor" x="418" y="51" text-anchor="middle">binds A</text></g> <g><rect stroke="currentColor" x="24" y="73" width="452" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="89" text-anchor="middle">both Tags registered: A and B exist</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="121" x2="244" y2="121" marker-end="url(#alc-seq-arrow-p49zoe)"></line><rect stroke="currentColor" fill="none" x="89.4" y="102" width="153.2" height="15" rx="3"></rect><text fill="currentColor" x="166" y="114" text-anchor="middle">precreate: reserve URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="161" x2="412" y2="161" marker-end="url(#alc-seq-arrow-p49zoe)"></line><rect stroke="currentColor" fill="none" x="173.4" y="142" width="153.2" height="15" rx="3"></rect><text fill="currentColor" x="250" y="154" text-anchor="middle">precreate: reserve URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="201" x2="244" y2="201" marker-end="url(#alc-seq-arrow-p49zoe)"></line><rect stroke="currentColor" fill="none" x="82.80000000000001" y="182" width="166.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="166" y="194" text-anchor="middle">create, B's URL deferred</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="241" x2="412" y2="241" marker-end="url(#alc-seq-arrow-p49zoe)"></line><rect stroke="currentColor" fill="none" x="166.8" y="222" width="166.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="250" y="234" text-anchor="middle">create, A's URL deferred</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="281" x2="244" y2="281" marker-end="url(#alc-seq-arrow-p49zoe)"></line><rect stroke="currentColor" fill="none" x="96" y="262" width="140" height="15" rx="3"></rect><text fill="currentColor" x="166" y="274" text-anchor="middle">update: wire B's URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="321" x2="412" y2="321" marker-end="url(#alc-seq-arrow-p49zoe)"></line><rect stroke="currentColor" fill="none" x="180" y="302" width="140" height="15" rx="3"></rect><text fill="currentColor" x="250" y="314" text-anchor="middle">update: wire A's URL</text></g></svg> <svg viewBox="-2 0 326 366" style="max-width: 375px" role="img" aria-label="Neither Worker waits for the other: each is reserved and created first, then a second pass wires in the real URLs."><defs><marker id="alc-seq-arrow-n-p49zoe" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-p49zoe" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="60" x2="52.333333333333336" y2="362"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="100.66666666666667" height="58" rx="8"></rect><text fill="currentColor" x="52.333333333333336" y="22" text-anchor="middle">alchemy</text> <text fill="currentColor" x="52.333333333333336" y="37" text-anchor="middle">deploy</text> <text fill="currentColor" x="52.333333333333336" y="52" text-anchor="middle">the plan</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="60" x2="161" y2="362"></line><rect stroke="currentColor" fill="none" x="110.66666666666666" y="2" width="100.66666666666667" height="58" rx="8"></rect><text fill="currentColor" x="161" y="22" text-anchor="middle">Worker A</text> <text fill="currentColor" x="161" y="52" text-anchor="middle">binds B</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="269.66666666666663" y1="60" x2="269.66666666666663" y2="362"></line><rect stroke="currentColor" fill="none" x="219.3333333333333" y="2" width="100.66666666666667" height="58" rx="8"></rect><text fill="currentColor" x="269.66666666666663" y="22" text-anchor="middle">Worker B</text> <text fill="currentColor" x="269.66666666666663" y="52" text-anchor="middle">binds A</text></g> <g><rect stroke="currentColor" x="2" y="74" width="318" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="161" y="90" text-anchor="middle">both Tags registered: A and B exist</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="122" x2="155" y2="122" marker-end="url(#alc-seq-arrow-n-p49zoe)"></line><rect stroke="currentColor" fill="none" x="30.066666666666677" y="103" width="153.2" height="15" rx="3"></rect><text fill="currentColor" x="106.66666666666667" y="115" text-anchor="middle">precreate: reserve URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="162" x2="263.66666666666663" y2="162" marker-end="url(#alc-seq-arrow-n-p49zoe)"></line><rect stroke="currentColor" fill="none" x="84.39999999999998" y="143" width="153.2" height="15" rx="3"></rect><text fill="currentColor" x="160.99999999999997" y="155" text-anchor="middle">precreate: reserve URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="202" x2="155" y2="202" marker-end="url(#alc-seq-arrow-n-p49zoe)"></line><rect stroke="currentColor" fill="none" x="23.466666666666683" y="183" width="166.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="106.66666666666667" y="195" text-anchor="middle">create, B's URL deferred</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="242" x2="263.66666666666663" y2="242" marker-end="url(#alc-seq-arrow-n-p49zoe)"></line><rect stroke="currentColor" fill="none" x="77.79999999999998" y="223" width="166.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="160.99999999999997" y="235" text-anchor="middle">create, A's URL deferred</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="282" x2="155" y2="282" marker-end="url(#alc-seq-arrow-n-p49zoe)"></line><rect stroke="currentColor" fill="none" x="36.66666666666667" y="263" width="140" height="15" rx="3"></rect><text fill="currentColor" x="106.66666666666667" y="275" text-anchor="middle">update: wire B's URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="322" x2="263.66666666666663" y2="322" marker-end="url(#alc-seq-arrow-n-p49zoe)"></line><rect stroke="currentColor" fill="none" x="90.99999999999997" y="303" width="140" height="15" rx="3"></rect><text fill="currentColor" x="160.99999999999997" y="315" text-anchor="middle">update: wire A's URL</text></g></svg>

Neither Worker waits for the other: each is reserved and created first, then a second pass wires in the real URLs.

1. The **Tags** are registered up front, so the graph knows that A and B both exist.
2. Each provider’s [`precreate`](../infrastructure-as-code/provider.md#precreate) hook reserves the resource (and its physical URL) without needing the other side’s outputs.
3. `create` runs in parallel using deferred Outputs — bindings see `Output<string>` placeholders that resolve later.
4. A converge pass calls `update` once both sides exist, wiring the real cross-references in. Types stay sound the whole way through because Outputs are typed.

The same pattern works for Lambda↔Lambda, Worker↔Container, or any mix — the Tag/Layer split is a property of every Function resource ([Runtime](runtime.md)), not just Workers.

## When you don’t need this

If your services form a DAG (no cycles), you can keep declaration and implementation in one expression:

```typescript
export default Cloudflare.Worker(
  "MyWorker",
  { main: import.meta.url },
  Effect.gen(function* () {
    /* ... */
  }),
);
```

The tagged-class pattern only pays off when something else needs to reference the Worker before its implementation is in scope. For non-circular reuse across files, see [Layers](layers.md).

The deferred `Output` mechanism behind the two-phase plan is covered in [Inputs & Outputs](../infrastructure-as-code/outputs.md).

## Where next

- [Inputs & Outputs](../infrastructure-as-code/outputs.md) — the deferred references that make the cycle resolvable.
- [Providers](../infrastructure-as-code/provider.md) — the `precreate` hook that reserves resources up front.
