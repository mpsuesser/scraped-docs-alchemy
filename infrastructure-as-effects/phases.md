---
url: https://alchemy.run/infrastructure-as-effects/phases
title: "Phases"
description: "Alchemy programs run in two phases — Construction drives the deploy, Runtime handles requests. Knowing which is which is the key to writing Workers and Lambda Functions."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Every alchemy program runs in two phases — plantime builds the plan, runtime serves requests. [Runtime](runtime.md) resources — Workers, Lambdas, Containers — express both in a single program by **returning an Effect from inside an Effect**. Earlier pages named the two halves the Construction phase and the Runtime phase. Here is exactly when each one runs.

## Construction vs Runtime

```typescript
Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    // ─── Construction phase ───
    const bucket = yield* Cloudflare.R2.ReadWriteBucket(Bucket);

    return {
      // ─── Runtime phase ───
      fetch: Effect.gen(function* () {
        const obj = yield* bucket.get("key");
        return HttpServerResponse.text(yield* obj.text());
      }),
    };
  }).pipe(Effect.provide(Cloudflare.R2.ReadWriteBucketBinding)),
);
```

| Phase | Code | When it runs |
| --- | --- | --- |
| Construction | outer | At plantime **and** at cold start |
| Runtime | inner | Only inside a deployed handler |

The `bucket` value is established once during Construction and captured by the runtime closure. The constructor runs at most once per cold start; the runtime body runs per request with everything already wired up. Each phase has its own `Scope` with very different lifetimes — [Instance scope vs request scope](runtime.md#instance-scope-vs-request-scope) covers where cleanup can (and cannot) happen.

The Runtime phase is the *only* place where `Alchemy.RuntimeContext` is available. Any Effect whose requirements include `RuntimeContext` can only execute inside the runtime closure — the type system rejects it everywhere else. [Layers](layers.md#the-types-hold-the-boundary) builds on this split to keep whole services honest about where they can run.

## What runs when

<svg viewBox="0 0 699.6 124" style="max-width: 805px" role="img" aria-label="Construction runs twice: at plantime, to record the bindings, and at cold start, to build real clients. The handler only ever runs per request."><line stroke-opacity="0.2" stroke="currentColor" x1="419.2666666666667" y1="24" x2="419.2666666666667" y2="118"></line><g><text fill="currentColor" x="4" y="50">alchemy deploy</text> <text fill="currentColor" x="4" y="64">plantime</text> <g><rect stroke="currentColor" fill="none" x="127.60000000000001" y="40" width="128.33333333333331" height="24" rx="5"></rect><text fill="currentColor" x="191.76666666666665" y="56" text-anchor="middle">Construction</text></g> <g><rect stroke="currentColor" fill="none" x="255.93333333333334" y="40" width="128.33333333333337" height="24" rx="5"></rect><text fill="currentColor" x="320.1" y="56" text-anchor="middle">create, update</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="699.6" y2="74"></line><text fill="currentColor" x="4" y="94">Worker</text> <text fill="currentColor" x="4" y="108">runtime</text> <g><rect stroke="currentColor" fill="none" x="419.2666666666667" y="84" width="128.33333333333331" height="24" rx="5"></rect><text fill="currentColor" x="483.4333333333334" y="100" text-anchor="middle">Construction</text></g> <g><rect stroke="currentColor" fill="none" x="559.2666666666667" y="84" width="52.500000000000114" height="24" rx="5"></rect><text fill="currentColor" x="585.5166666666667" y="100" text-anchor="middle">fetch</text></g> <g><rect stroke="currentColor" fill="none" x="623.4333333333334" y="84" width="52.5" height="24" rx="5"></rect><text fill="currentColor" x="649.6833333333334" y="100" text-anchor="middle">fetch</text></g></g> <text fill="currentColor" x="419.2666666666667" y="18" text-anchor="middle">cold start</text></svg> <svg viewBox="0 0 330 174" style="max-width: 379px" role="img" aria-label="Construction runs twice: at plantime, to record the bindings, and at cold start, to build real clients. The handler only ever runs per request."><line stroke-opacity="0.2" stroke="currentColor" x1="165.625" y1="24" x2="165.625" y2="168"></line><g><text fill="currentColor" x="0" y="44">alchemy deploy <tspan fill="currentColor" dx="8">plantime</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="72.875" height="24" rx="5"></rect><text fill="currentColor" x="0" y="90" text-anchor="start">Construction</text></g> <g><rect stroke="currentColor" fill="none" x="72.875" y="54" width="72.875" height="24" rx="5"></rect><text fill="currentColor" x="151.75" y="70" text-anchor="start">create, update</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="99" x2="330" y2="99"></line><text fill="currentColor" x="0" y="113">Worker <tspan fill="currentColor" dx="8">runtime</tspan></text> <g><rect stroke="currentColor" fill="none" x="165.625" y="123" width="72.875" height="24" rx="5"></rect><text fill="currentColor" x="165.625" y="159" text-anchor="start">Construction</text></g> <g><rect stroke="currentColor" fill="none" x="245.125" y="123" width="29.812500000000057" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="281.5625" y="123" width="29.8125" height="24" rx="5"></rect><text fill="currentColor" x="281.5625" y="159" text-anchor="start">fetch</text></g></g> <text fill="currentColor" x="165.625" y="18" text-anchor="middle">cold start</text></svg>

Construction: records bindings, then builds clientshandler, per request

Construction runs twice: at plantime, to record the bindings, and at cold start, to build real clients. The handler only ever runs per request.

- At **plantime**, the constructor runs to discover bindings — alchemy needs to know which resources the handler will use so it can wire permissions, env vars, and references.
- At **runtime cold start**, the constructor runs again — this time inside the deployed Worker, where the same `bind()` calls return live SDK clients backed by the deployed resource.
- The **runtime body** only runs in the deployed handler. It never executes at plantime, so you can put real per-request work there without affecting deploy speed.

## ALCHEMY\_PHASE

The current phase is exposed as the `ALCHEMY_PHASE` environment variable / config key:

| Value | Context |
| --- | --- |
| `plan` | Default. Running `alchemy deploy`, `alchemy plan`, or `alchemy dev`. |
| `runtime` | Running inside a deployed Worker or Lambda Function. |

Most user code never reads this directly — but providers and [Bindings](binding.md) use it internally to behave differently across phases.

## ALCHEMY\_DEV

`alchemy dev` is a plantime phase, so `ALCHEMY_PHASE` is `plan` during local development just like a regular deploy. To tell whether you’re running under `alchemy dev` specifically, read the `ALCHEMY_DEV` config key. The CLI sets `ALCHEMY_DEV=true` on the dev process; every other entrypoint leaves it unset, so it defaults to `false`.

```typescript
import { ALCHEMY_DEV } from "alchemy";

Effect.gen(function* () {
  if (yield* ALCHEMY_DEV) {
    // local-dev-only behavior
  }
});
```

## The \_\_ALCHEMY\_RUNTIME\_\_ guard

A binding’s contract is a `Binding.Service`; the guarded setup Effect lives in the implementation Layer you provide, not the contract. That setup Effect does two things — and the phase decides how much of it runs:

| Work | Phase | Job |
| --- | --- | --- |
| typed client | both | The lightweight typed SDK that ships with the bundle. |
| `host.bind(...)` wiring | plan | Deploy-time registration of IAM / env / native bindings. |

The deploy-time wiring is fenced behind `if (!globalThis.__ALCHEMY_RUNTIME__)`. At plantime the guard is open, so calling `bind()` records what the function will need. At runtime the global flag is set, so the guard is skipped and `bind()` returns just the typed client. The runtime bundle stays small because the planning branch never executes.

Which wiring the guard protects is decided by the implementation Layer you provided ([Bindings](binding.md#a-contract-and-a-layer)).

This is also how alchemy can let you write `bucket.get(...)` inside a Worker without bundling AWS / Cloudflare provisioning code: the provisioning lives behind the guard, never on the runtime path.

## Why this matters

The Construction/Runtime split lets you write code that:

1. **Resolves infrastructure references at deploy time** — bindings know which bucket ARN, queue URL, etc. to inject.
2. **Initializes SDK clients once at cold start** — not on every request.
3. **Handles requests with a pre-configured context** — the `bucket` variable in the runtime body already knows which resource to talk to.

## Where next

- [Circular Bindings](circular-bindings.md) — the two-pass plan this phase split makes possible. Next page.
- [Layers](layers.md) — `RuntimeContext` as a type-level boundary; infrastructure behind service interfaces.
- [State Store](../state-store.md) — where deploy results persist between runs.
