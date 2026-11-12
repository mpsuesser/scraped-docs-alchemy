---
url: https://alchemy.run/cloudflare/data/kv
title: "KV"
description: "How Workers KV replicates values to every city for fast reads, what eventual consistency means for writes, and how Workers read, write, list and expire keys through access-scoped bindings."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Workers KV is a key-value store copied to every city in Cloudflare’s network. A Worker reads a key from the copy nearest the request, so reads are fast everywhere. Writes go to a central store and spread out from there.

## Fast reads, eventual writes

A value written in one city reaches the others within about a minute. Until it does, a read elsewhere can return the old value, or nothing.

<svg viewBox="0 0 655.2 186" style="max-width: 753px" role="img" aria-label="A write is read back at once where it was made. Other cities see it within about a minute, and until then they read the old value."><line stroke-opacity="0.2" stroke="currentColor" x1="83.2" y1="24" x2="83.2" y2="162"></line><g><text fill="currentColor" x="4" y="50">Paris</text> <text fill="currentColor" x="4" y="64">writes</text> <g><rect stroke="currentColor" fill="none" x="83.2" y="40" width="560" height="24" rx="5"></rect><text fill="currentColor" x="363.2" y="56" text-anchor="middle">reads v2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="655.2" y2="74"></line><text fill="currentColor" x="4" y="100">Tokyo</text> <g><rect stroke="currentColor" fill="none" x="83.2" y="84" width="205.33333333333331" height="24" rx="5"></rect><text fill="currentColor" x="185.86666666666667" y="100" text-anchor="middle">still reads v1</text></g> <g><rect stroke="currentColor" fill="none" x="288.5333333333333" y="84" width="354.66666666666674" height="24" rx="5"></rect><text fill="currentColor" x="465.8666666666667" y="100" text-anchor="middle">reads v2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="655.2" y2="118"></line><text fill="currentColor" x="4" y="144">New York</text> <g><rect stroke="currentColor" fill="none" x="83.2" y="128" width="382.6666666666667" height="24" rx="5"></rect><text fill="currentColor" x="274.53333333333336" y="144" text-anchor="middle">still reads v1</text></g> <g><rect stroke="currentColor" fill="none" x="465.8666666666667" y="128" width="177.33333333333337" height="24" rx="5"></rect><text fill="currentColor" x="554.5333333333333" y="144" text-anchor="middle">reads v2</text></g></g> <text fill="currentColor" x="83.2" y="18" text-anchor="middle">put v2</text> <line stroke-opacity="0.2" stroke="currentColor" x1="83.2" y1="162" x2="643.2" y2="162"></line><text fill="currentColor" x="83.2" y="178" text-anchor="start">0s</text> <text fill="currentColor" x="363.2" y="178" text-anchor="middle">30s</text> <text fill="currentColor" x="643.2" y="178" text-anchor="end">about a minute</text></svg> <svg viewBox="0 0 330 216" style="max-width: 379px" role="img" aria-label="A write is read back at once where it was made. Other cities see it within about a minute, and until then they read the old value."><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="24" x2="0" y2="192"></line><g><text fill="currentColor" x="0" y="44">Paris <tspan fill="currentColor" dx="8">writes</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="70" text-anchor="middle">reads v2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">Tokyo</text> <g><rect stroke="currentColor" fill="none" x="0" y="108" width="116.6" height="24" rx="5"></rect><text fill="currentColor" x="58.3" y="124" text-anchor="middle">still reads v1</text></g> <g><rect stroke="currentColor" fill="none" x="116.6" y="108" width="201.4" height="24" rx="5"></rect><text fill="currentColor" x="217.3" y="124" text-anchor="middle">reads v2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">New York</text> <g><rect stroke="currentColor" fill="none" x="0" y="162" width="217.3" height="24" rx="5"></rect><text fill="currentColor" x="108.65" y="178" text-anchor="middle">still reads v1</text></g> <g><rect stroke="currentColor" fill="none" x="217.3" y="162" width="100.69999999999999" height="24" rx="5"></rect><text fill="currentColor" x="267.65" y="178" text-anchor="middle">reads v2</text></g></g> <text fill="currentColor" x="0" y="18" text-anchor="start">put v2</text> <line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="192" x2="318" y2="192"></line><text fill="currentColor" x="0" y="208" text-anchor="start">0s</text> <text fill="currentColor" x="159" y="208" text-anchor="middle">30s</text> <text fill="currentColor" x="318" y="208" text-anchor="end">about a minute</text></svg>

A write is read back at once where it was made. Other cities see it within about a minute, and until then they read the old value.

So KV fits data that’s read far more often than it’s written, where a moment of staleness is fine: configuration, feature flags, sessions, cached API responses, redirects. For relational data, use [D1](d1.md). For state that must be consistent the instant it’s written, use a [Durable Object](../compute/durable-objects.md).

## Declare a namespace

Keys live in a **namespace**:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Cache = Cloudflare.KV.Namespace("Cache");
```

Alchemy names it from your app, stage and id, so each stage gets its own.

## Read and write values

Bind the namespace in a Worker and get a typed client. `ReadWriteNamespace` gives it both reads and writes:

```typescript
export default Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    const kv = yield* Cloudflare.KV.ReadWriteNamespace(Cache);
    return {
      fetch: Effect.gen(function* () {
        yield* kv.put("greeting", "hello");
        const value = yield* kv.get("greeting");
        return HttpServerResponse.text(value ?? "not found");
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(Cloudflare.KV.ReadWriteNamespaceBinding)),
);
```

A missing key reads as `null`. Every operation can fail with a `NamespaceError`, which stays in the type until you handle it.

## Read typed values

`get` returns text by default. Ask for JSON and give its type, or read several keys at once:

```typescript
const config = yield* kv.get<Config>("config", "json");
const many = yield* kv.get(["a", "b"], "text");
```

`"arrayBuffer"` and `"stream"` read binary values.

## Expire keys and attach metadata

A key can expire on its own, after at least 60 seconds. It can also carry metadata, read back with the value in one call:

```typescript
yield* kv.put(\`session:${id}\`, token, {
  expirationTtl: 3600,
  metadata: { userId },
});

const { value, metadata } = yield* kv.getWithMetadata<{ userId: string }>(\`session:${id}\`);
```

## List keys

`list` returns keys a page at a time, optionally those starting with a prefix:

```typescript
const page = yield* kv.list({ prefix: "session:" });
const names = page.keys.map((key) => key.name);
```

Each key comes with its metadata. Put what you need to show in a list into metadata, and you can render the list without reading every value.

## Ask for only the access you need

The client comes in three access levels. `ReadNamespace` can `get`, `getWithMetadata` and `list`. `WriteNamespace` can `put` and `delete`. `ReadWriteNamespace` can do both. A Worker that only serves configuration should bind `ReadNamespace`, so its code can’t change it.

Each level has two implementations, chosen by the Layer the Worker provides. The `*NamespaceBinding` Layers use a native KV binding on the Worker. The `*NamespaceHttp` Layers use Cloudflare’s API instead, with an API token Alchemy creates for just that namespace’s access, for code running outside a Worker. The handlers don’t change between them.
