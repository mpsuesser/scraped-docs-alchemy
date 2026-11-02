---
url: https://alchemy.run/fly/data/tigris
title: "Tigris"
description: "S3-compatible object storage billed through Fly. Bind Fly.PutObject / Fly.GetObject."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A [`Fly.Bucket`](https://alchemy.run/providers/fly/reference/bucket#bucket) is Tigris S3-compatible
object storage, billed through Fly. It is org-scoped. There is no
parent [App](../compute/apps.md).

Tigris speaks the S3 API. Bind [`PutObject`](https://alchemy.run/providers/fly/reference/bucket#putobject)
/ [`GetObject`](https://alchemy.run/providers/fly/reference/bucket#getobject) in a
[Service](../compute/services.md). Alchemy injects the Tigris
endpoint and credentials. You never read `AWS_*` yourself.

## Create a Bucket

Alchemy generates a unique name unless you pass one.

```typescript
const data = yield* Fly.Bucket("Data");
```

## A stable name

Pass `name` when you need a stable Tigris bucket name.

```typescript
const data = yield* Fly.Bucket("Data", {
  name: "my-bucket",
});
```

:::caution[Changing `name` replaces the Bucket]
Tigris cannot rename a bucket. Alchemy creates the new name, then
deletes the old one.
:::

## Put and get objects

Bind `PutObject` / `GetObject` in the Service's constructor. Provide the matching
`*Http` layers.

```typescript
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends Fly.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const putObject = yield* Fly.PutObject(Data);
    const getObject = yield* Fly.GetObject(Data);

    return {
      fetch: Effect.gen(function* () {
        yield* putObject({
          Key: "hello.txt",
          Body: "hello",
          ContentType: "text/plain",
        });
        const obj = yield* getObject({ Key: "hello.txt" });
        return HttpServerResponse.json({ ok: obj.ETag !== undefined });
      }),
    };
  }).pipe(Effect.provide([Fly.PutObjectHttp, Fly.GetObjectHttp])),
) {}
```

## Public access

Buckets are private by default. `public: true` serves objects
without credentials.

```typescript
const assets = yield* Fly.Bucket("Assets", {
  public: true,
});
```

## Custom domain

`domainName` sets `website.domain_name` on the add-on. Point a
CNAME at `{name}.t3.tigrisbucket.io`.

```typescript
const assets = yield* Fly.Bucket("Assets", {
  public: true,
  domainName: "assets.example.com",
});
```

## Organization

Org defaults to the current token. Pass `orgSlug` to pin it.

```typescript
const data = yield* Fly.Bucket("Data", {
  orgSlug: "my-org",
});
```

:::caution[Changing `orgSlug` replaces the Bucket]
The bucket is created in the new org. The old bucket is deleted.
:::

## Where next

[Services](../compute/services.md) is the Machine that binds
`PutObject` / `GetObject`.
The [`Bucket` reference](https://alchemy.run/providers/fly/reference/bucket#bucket) lists every prop.
