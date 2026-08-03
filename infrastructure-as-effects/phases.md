---
url: https://alchemy.run/infrastructure-as-effects/phases
title: "Phases"
description: "Alchemy programs run in two phases — plantime/init drives the deploy, runtime handles requests. Knowing which is which is the key to writing Workers and Lambda Functions."
access_date: 2026-08-03T19:43:15.086Z
current_date: 2026-08-03T19:43:15.086Z
---

Every alchemy program runs in two phases — plantime builds the plan, runtime serves requests. Function resources — Workers, Lambdas, Containers — express both in a single program by **returning an Effect from inside an Effect**.

## Init vs runtime

```typescript
Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    // ─── Init phase ───
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
| Init | outer | At plantime **and** at cold start |
| Runtime | inner | Only inside a deployed handler |

The `bucket` value is established once during init and captured by the runtime closure. Init runs at most once per cold start; the runtime body runs per request with everything already wired up. Each phase has its own `Scope` with very different lifetimes — [Instance scope vs request scope](functions-and-servers.md#instance-scope-vs-request-scope) covers where cleanup can (and cannot) happen.

The runtime phase is the *only* place where `Alchemy.RuntimeContext` is available. Any Effect whose requirements include `RuntimeContext` can only execute inside the runtime closure — the type system rejects it everywhere else. The next page builds the [colored-function model](layers.md#runtime-as-a-colored-function) on top of this split.

## What runs when

<svg viewBox="0 0 848 204" role="img" aria-label="Plantime (top) records bindings and builds the plan. Runtime (bottom) starts on cold start and runs the handler per request."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 56 C 201 56, 201 56, 238 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 384 56 C 421 56, 421 56, 458 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 604 56 C 641 56, 641 102, 678 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 164 148 C 201 148, 201 148, 238 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 384 148 C 421 148, 421 148, 458 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">alchemy deploy</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">plantime</text></g> <g transform="translate(244, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">init()</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">records bindings</text></g> <g transform="translate(464, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">apply</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">create / update</text></g> <g transform="translate(684, 70)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">deployed</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Worker / Lambda</text></g> <g transform="translate(24, 116)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">cold start</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">runtime</text></g> <g transform="translate(244, 116)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">init()</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">builds SDK clients</text></g> <g transform="translate(464, 116)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">fetch()</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">per request</text></g></svg>

Plantime (top) records bindings and builds the plan. Runtime (bottom) starts on cold start and runs the handler per request.

- At **plantime**, init runs to discover bindings — alchemy needs to know which resources the handler will use so it can wire permissions, env vars, and references.
- At **runtime cold start**, init runs again — this time inside the deployed Worker, where the same `bind()` calls return live SDK clients backed by the deployed resource.
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

The init/runtime split lets you write code that:

1. **Resolves infrastructure references at deploy time** — bindings know which bucket ARN, queue URL, etc. to inject.
2. **Initializes SDK clients once at cold start** — not on every request.
3. **Handles requests with a pre-configured context** — the `bucket` variable in the runtime body already knows which resource to talk to.

Next: [Layers](layers.md) turns this phase split into a type-level model — `RuntimeContext` as a colored function.

## Where next

- [Layers](layers.md) — `RuntimeContext` as a colored function; infrastructure behind service interfaces.
- [State Store](../state-store.md) — where deploy results persist between runs.
