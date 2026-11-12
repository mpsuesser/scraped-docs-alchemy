---
url: https://alchemy.run/cloudflare/observability/workers-tracing
title: "Native Workers tracing"
description: "Mirror Effect spans into Cloudflare Workers Observability with Cloudflare.Telemetry() — the same Layer shape as Axiom.Telemetry, with Cloudflare owning sampling and export."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Cloudflare Workers can emit OpenTelemetry traces natively: fetch, KV, R2, and D1 are auto-instrumented, and custom spans join that waterfall via `tracing.startActiveSpan`. Alchemy’s `Cloudflare.Telemetry()` Layer forwards `Effect.withSpan` / `Effect.fn` frames into that API so Effect code shows up next to the platform spans.

## Provide the layer

Pin a compatibility date at or after `2026-07-28` (`startActiveSpan` GA). The global Worker default is still older, so omit the date and deploy fails instead of silently dropping Effect spans.

```typescript
export default Cloudflare.Worker(
  "Worker",
  {
    main: import.meta.url,
    compatibility: { date: "2026-08-25" },
  },
  Effect.gen(function* () {
    return {
      fetch: Effect.gen(function* () {
        yield* doWork().pipe(Effect.withSpan("operation"));
        return HttpServerResponse.text("ok");
      }),
    };
  }).pipe(
    Effect.provide(
      Layer.mergeAll(
        Cloudflare.R2.ReadWriteBucketBinding,
        Cloudflare.Telemetry(),
      ),
    ),
  ),
);
```

You do **not** set `observability.traces.enabled` on the Worker, write a Wrangler `observability` block, or configure an OTLP URL. The Layer enables traces on the host (the same bind path as `Cloudflare.cache()`) and installs a per-event Effect Tracer. Cloudflare owns sampling and export — spans submit with the request, no `waitUntil` flush.

Existing `Effect.withSpan` / `Effect.fn` instrumentation does not change.

During `alchemy dev` the compatibility date is not gated and the Layer has no deploy-time effect; whether spans are recorded locally is up to the local runtime. A Worker opted out of local emulation with `Alchemy.remote()` traces normally.

## What you get

### Effect spans in the platform waterfall

`Effect.withSpan` frames nest with Cloudflare’s own spans. A KV read inside a span shows up as its child, even when the fiber that issued it was forked or resumed on a later tick.

```typescript
const handle = Effect.gen(function* () {
  const user = yield* kv.get("user:1").pipe(Effect.withSpan("load.user"));
  yield* Effect.all([enrich(user), audit(user)], { concurrency: "unbounded" });
}).pipe(Effect.withSpan("handle"));
```

```text
http.server GET
└─ handle
   ├─ load.user
   │  └─ kv_get
   ├─ enrich
   └─ audit
```

<svg viewBox="0 0 707 278" style="max-width: 813px" role="img" aria-label="The same trace as a waterfall. Cloudflare's own spans, like kv_get, nest under your Effect spans, and enrich and audit run side by side."><g><text fill="currentColor" x="4" y="34">http.server GET</text> <g><rect stroke="currentColor" fill="none" x="135" y="18" width="560" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="707" y2="52"></line><text fill="currentColor" x="4" y="78">&nbsp;└ handle</text> <g><rect stroke="currentColor" fill="none" x="151.8" y="62" width="520.8" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="707" y2="96"></line><text fill="currentColor" x="4" y="122">&nbsp;└ load.user</text> <g><rect stroke="currentColor" fill="none" x="163" y="106" width="207.20000000000005" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="140" x2="707" y2="140"></line><text fill="currentColor" x="4" y="166">&nbsp;└ kv_get</text> <g><rect stroke="currentColor" fill="none" x="179.8" y="150" width="173.59999999999997" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="184" x2="707" y2="184"></line><text fill="currentColor" x="4" y="210">&nbsp;└ enrich</text> <g><rect stroke="currentColor" fill="none" x="381.40000000000003" y="194" width="268.7999999999999" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="228" x2="707" y2="228"></line><text fill="currentColor" x="4" y="254">&nbsp;└ audit</text> <g><rect stroke="currentColor" fill="none" x="381.40000000000003" y="238" width="151.19999999999987" height="24" rx="5"></rect></g></g></svg> <svg viewBox="0 0 330 338" style="max-width: 379px" role="img" aria-label="The same trace as a waterfall. Cloudflare's own spans, like kv_get, nest under your Effect spans, and enrich and audit run side by side."><g><text fill="currentColor" x="0" y="22">http.server GET</text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="318" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">&nbsp;└ handle</text> <g><rect stroke="currentColor" fill="none" x="9.54" y="86" width="295.73999999999995" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">&nbsp;└ load.user</text> <g><rect stroke="currentColor" fill="none" x="15.9" y="140" width="117.66" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="170" x2="330" y2="170"></line><text fill="currentColor" x="0" y="184">&nbsp;└ kv_get</text> <g><rect stroke="currentColor" fill="none" x="25.44" y="194" width="98.58000000000001" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="224" x2="330" y2="224"></line><text fill="currentColor" x="0" y="238">&nbsp;└ enrich</text> <g><rect stroke="currentColor" fill="none" x="139.92000000000002" y="248" width="152.64" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="278" x2="330" y2="278"></line><text fill="currentColor" x="0" y="292">&nbsp;└ audit</text><g><rect stroke="currentColor" fill="none" x="139.92000000000002" y="302" width="85.85999999999999" height="24" rx="5"></rect></g></g></svg>

The same trace as a waterfall. Cloudflare's own spans, like kv\_get, nest under your Effect spans, and enrich and audit run side by side.

### Attributes

Scalar annotations (`string`, `number`, `boolean`) are forwarded to the Cloudflare span. Nested objects, span events, and links stay Effect-local.

```typescript
yield* Effect.annotateCurrentSpan("user.id", userId);   // forwarded
yield* Effect.annotateCurrentSpan("retries", 2);        // forwarded
yield* Effect.annotateCurrentSpan("cached", true);      // forwarded
yield* Effect.annotateCurrentSpan("input", { nested }); // Effect-local
```

### Outcome

Completion is recorded as the `effect.exit` attribute: `success`, `failure`, or `interrupted`. A failure also sets the span status to `error`, with the first error’s message, so OpenTelemetry destinations mark the span as failed. Success and interruption leave the status `unset`. The Workers Observability dashboard doesn’t show span status yet, so filter on `effect.exit` there.

```typescript
yield* Effect.fail(new Error("boom")).pipe(
  Effect.withSpan("failing"), // effect.exit = "failure", status = error
  Effect.ignore,
);
const fiber = yield* Effect.forkChild(
  Effect.sleep("30 seconds").pipe(Effect.withSpan("slow")),
);
yield* Fiber.interrupt(fiber); // effect.exit = "interrupted"
```

### Sampling and persistence

`headSamplingRate` and `persist` are passed straight through to the Worker’s `observability.traces`. Omit them to keep Cloudflare’s defaults. Cloudflare applies the sampling rate when it ingests the trace, so every invocation still sees a live `Tracer`.

```typescript
Cloudflare.Telemetry({
  headSamplingRate: 0.1, // keep 10% of invocations
  persist: false,        // export only, nothing in the dashboard
})
```

## Combining with other exporters

Effect has a single `Tracer`, and `Cloudflare.Telemetry()` provides it. Any other exporter you merge in keeps its loggers and metric exporters; only its `traces` are superseded. With Axiom, for example, leave `traces` off and the logs and metrics still flow, in either `Layer.mergeAll` order:

```typescript
Effect.provide(
  Layer.mergeAll(
    Cloudflare.Telemetry(),
    Axiom.Telemetry({ token: Ingest, logs: Logs, metrics: Metrics }),
  ),
)
```

The same goes for the generic `Alchemy.Telemetry.layerOtlp({ logs, metrics })`.

To ship the waterfall (Effect + platform spans) to another backend, export it from Cloudflare with an [`ObservabilityDestination`](axiom-observability.md) rather than sending a second Worker-side OTLP trace stream.

## Async Workers

Non-Effect Workers keep the `observability` prop — there is no Tracer to install:

```typescript
yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  observability: {
    traces: { enabled: true },
  },
});
```
