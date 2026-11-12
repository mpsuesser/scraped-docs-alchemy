---
url: https://alchemy.run/cloudflare/compute/cache
title: "Workers Cache"
description: "Serve responses from Cloudflare's edge cache before your Worker even runs — enable it with one yield, control it with standard headers, purge it by tag from inside a handler."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Workers Cache sits in front of a Worker: when it’s enabled, Cloudflare checks the cache before invoking the Worker at all, and a hit is served straight from the edge. What gets cached is controlled by standard response headers: `Cache-Control` (including `stale-while-revalidate`), `Cache-Tag` for tag-based purging, and `Vary` for content negotiation.

<svg viewBox="0 0 670 124" style="max-width: 770px" role="img" aria-label="With Cache-Control: max-age=300, the first request runs the Worker and the response is stored. Every request for the next five minutes is a hit, and the Worker doesn't run at all."><line stroke-opacity="0.2" stroke="currentColor" x1="109.2" y1="24" x2="109.2" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="512.4" y1="24" x2="512.4" y2="118"></line><g><text fill="currentColor" x="4" y="50">GET /posts</text> <text fill="currentColor" x="4" y="64">requests</text> <g><rect stroke="currentColor" fill="none" x="109.2" y="40" width="22.39999999999999" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="176.39999999999998" y="40" width="16.80000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="243.6" y="40" width="16.799999999999983" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="310.8" y="40" width="16.80000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="389.2" y="40" width="16.80000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="512.4" y="40" width="22.399999999999977" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="579.5999999999999" y="40" width="16.800000000000182" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="670" y2="74"></line><text fill="currentColor" x="4" y="94">Api</text> <text fill="currentColor" x="4" y="108">Worker</text> <g><rect stroke="currentColor" fill="none" x="109.2" y="84" width="50.40000000000002" height="24" rx="5"></rect><text fill="currentColor" x="134.4" y="100" text-anchor="middle">runs</text></g> <g><rect stroke="currentColor" fill="none" x="512.4" y="84" width="50.40000000000009" height="24" rx="5"></rect><text fill="currentColor" x="537.6" y="100" text-anchor="middle">runs</text></g></g> <text fill="currentColor" x="109.2" y="18" text-anchor="middle">miss</text> <text fill="currentColor" x="512.4" y="18" text-anchor="middle">5 minutes later: miss</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="With Cache-Control: max-age=300, the first request runs the Worker and the response is stored. Every request for the next five minutes is a hit, and the Worker doesn't run at all."><line stroke-opacity="0.2" stroke="currentColor" x1="6.36" y1="24" x2="6.36" y2="138"></line><line stroke-opacity="0.2" stroke="currentColor" x1="235.32" y1="24" x2="235.32" y2="138"></line><g><text fill="currentColor" x="0" y="44">GET /posts <tspan fill="currentColor" dx="8">requests</tspan></text> <g><rect stroke="currentColor" fill="none" x="6.36" y="54" width="12.719999999999999" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="44.519999999999996" y="54" width="9.54" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="82.68" y="54" width="9.539999999999992" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="120.84" y="54" width="9.539999999999992" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="165.36" y="54" width="9.539999999999992" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="235.32" y="54" width="12.720000000000027" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="273.48" y="54" width="9.539999999999964" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">Api <tspan fill="currentColor" dx="8">Worker</tspan></text> <g><rect stroke="currentColor" fill="none" x="6.36" y="108" width="28.620000000000005" height="24" rx="5"></rect><text fill="currentColor" x="40.980000000000004" y="124" text-anchor="start">runs</text></g> <g><rect stroke="currentColor" fill="none" x="235.32" y="108" width="28.620000000000005" height="24" rx="5"></rect><text fill="currentColor" x="269.94" y="124" text-anchor="start">runs</text></g></g> <text fill="currentColor" x="6.36" y="18" text-anchor="start">miss</text> <text fill="currentColor" x="235.32" y="18" text-anchor="middle">5 minutes later: miss</text></svg>

miss: the Worker answershit: served from the cache

With Cache-Control: max-age=300, the first request runs the Worker and the response is stored. Every request for the next five minutes is a hit, and the Worker doesn't run at all.

## Enable the cache on a Worker

`yield*` `Cloudflare.cache()` in the Worker’s Construction phase. It turns Workers Cache on at deploy time and returns the runtime client. There is no layer to provide:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default Cloudflare.Worker(
  "CachedWorker",
  { main: import.meta.url },
  Effect.gen(function* () {
    const { purge } = yield* Cloudflare.cache();

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;

        if (request.url.startsWith("/invalidate")) {
          yield* purge({ tags: ["products"] }).pipe(Effect.orDie);
          return HttpServerResponse.text("purged");
        }

        return HttpServerResponse.text("hello", {
          headers: {
            "Cache-Control": "public, max-age=300",
            "Cache-Tag": "products",
          },
        });
      }),
    };
  }),
);
```

Responses that carry a cacheable `Cache-Control` header are stored at the edge; for the next five minutes, repeat requests to the same URL never reach the handler.

## Purge by tag

The `purge` client drives the runtime `ctx.cache` API from inside a handler. Purge cached responses by the `Cache-Tag` values they were served with, by path prefix, or wholesale:

```typescript
yield* purge({ tags: ["products"] });
```

Failures surface as a typed `CachePurgeError`:

```typescript
const result = yield* purge({ tags: ["products"] }).pipe(
  Effect.catchTag("CachePurgeError", (error) =>
    Effect.succeed({ success: false, errors: [error.message] }),
  ),
);
```

By default the cache is scoped to a single Worker version, so every deploy starts cold. Opt into sharing cached responses across versions when your responses don’t depend on the code that produced them:

```typescript
const { purge } = yield* Cloudflare.cache({ crossVersionCache: true });
```

`enabled: false` keeps the binding’s purge client available while turning the read-through cache off, which is useful for toggling cache behavior per stage.

## Enable the cache with the cache prop

Workers whose `main` module exports a plain async `fetch` handler enable the cache with the `cache` prop instead:

```typescript
export const Worker = Cloudflare.Worker("Worker", {
  main: "./src/worker.ts",
  cache: { enabled: true, crossVersionCache: true },
});
```

The handler shapes cache behavior the same way, through `Cache-Control`, `Cache-Tag`, and `Vary` headers on its responses.

## Where next

- [Workers](workers.md) — the two-phase Worker model the cache fronts.
- [cache API reference](https://alchemy.run/providers/cloudflare/workers#cache) — options and purge types.
