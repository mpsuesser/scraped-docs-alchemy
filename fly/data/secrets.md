---
url: https://alchemy.run/fly/data/secrets
title: "Secrets"
description: "Config.Redacted on a Service, Fly.Secret when Fly should own the value, and KMS keys."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

Most secrets in a [`Service`](../compute/services.md) come from your
`.env`. Yield `Config.Redacted` in the constructor. Alchemy binds the value
onto the Machine.

Use [`Fly.Secret`](https://alchemy.run/providers/fly/reference/secret#secret) when the value should be
shared across Machines and managed in one place by Fly. Use
[`SecretKey`](https://alchemy.run/providers/fly/reference/secret-key#secretkey) for Fly KMS.

## Bind from `.env`

```typescript
import * as Config from "effect/Config";
import * as Redacted from "effect/Redacted";

export default class Api extends Fly.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const apiKey = yield* Config.Redacted("API_KEY");

    return {
      fetch: Effect.gen(function* () {
        const token = Redacted.value(apiKey);
        // ...
      }),
    };
  }),
) {}
```

`Config.Redacted("API_KEY")` reads `API_KEY` from the env of whoever
runs the deploy and writes it onto the Machine. At runtime the same
line resolves from that env var.

The value is `Redacted<string>`. Unwrap with `Redacted.value` only
where you need the raw string. Do not pass `env: { ... }` on a
Service. Yield Config.

See [Secrets & Config](../../environments/secrets.md) for combinators and
stages.

## When Fly should own the secret

`Fly.Secret` is the App vault. Fly injects it as an environment
variable on every Machine in a [`Fly.App`](../compute/apps.md). Use it
when the value is shared and managed in one place, not when only this
Service reads a deploy-time `.env` key.

```typescript
import * as Fly from "@alchemy.run/fly";
import * as Redacted from "effect/Redacted";

export const Site = Fly.App("Site");

export const ApiToken = Fly.Secret("ApiToken", {
  app: Site,
  name: "API_TOKEN",
  value: Redacted.make("…"),
});
```

A `Fly.Secret` targets a `Fly.App`, not a Service's own App. Run the
Services that read it in that App with `app: Site` — see
[Group Services in one App](../compute/services.md#group-services-in-one-app).

`name` is the env-var Machines see. Omit it and Alchemy generates an
ownership-stamped name. Updating `value` is in place. Changing `app`
or `name` replaces.

The resolved resource exposes `name`, `digest`, `createdAt`, and
`updatedAt`. Never the plaintext.

## Get, list, and write

`GetSecret` is bound to one Secret. `ListSecrets` is bound to the
App. The Service runs in the Secret's App:

```typescript
export default class Api extends Fly.Service<Api>()(
  "Api",
  { app: Site, main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const get = yield* Fly.GetSecret(ApiToken);

    return {
      fetch: Effect.gen(function* () {
        const got = yield* get().pipe(Effect.orDie);
        return HttpServerResponse.json({ name: got.name });
      }),
    };
  }).pipe(Effect.provide(Fly.GetSecretHttp)),
) {}
```

```typescript
const list = yield* Fly.ListSecrets(Site);
const { secrets } = yield* list();
```

From an Action, the org token can list any App in the org. From a
Machine, deploy tokens are per-App. Mixing Apps on one Machine
shares one `FLY_API_TOKEN` and is not supported.

Fly only returns plaintext from a Machine in the same App. From
deploy-time Actions you get metadata (name, digest, timestamps).

Write from an Action with `WriteSecret`:

```typescript
const Seed = Alchemy.Action(
  "Seed",
  Effect.gen(function* () {
    const secrets = yield* Fly.WriteSecret(ApiToken);

    return Effect.fn(function* () {
      yield* secrets.update("API_TOKEN", Redacted.make("sk_live_rotated"));
    });
  }).pipe(Effect.provide(Fly.WriteSecretHttp)),
);
```

Inside a Service, Alchemy mints an App deploy token for the Machine.
Inside an Action, the ambient `FLY_API_TOKEN` is used instead.

## KMS keys

[`SecretKey`](https://alchemy.run/providers/fly/reference/secret-key#secretkey) is an App KMS key, not an
env secret. Generate a random key or set raw material. Private
bytes never appear in attributes. Like `Fly.Secret`, it targets a
`Fly.App`; bind it from a Service that runs in that App (`app: Site`).

```typescript
export const Signing = Fly.SecretKey("Signing", {
  app: Site,
  type: "nacl_sign",
});

export const Box = Fly.SecretKey("Box", {
  app: Site,
  type: "nacl_secretbox",
});
```

`type` is Fly's key type (`nacl_sign`, `nacl_box`, `nacl_secretbox`,
`hs256`, `es256`, `xaes256gcm`, …). Changing `app`, `name`, or
`type` replaces the key.

## Encrypt, Decrypt, Sign, Verify

One binding per operation. The key is fixed when you bind.

```typescript
const encrypt = yield* Fly.Encrypt(Box);
const decrypt = yield* Fly.Decrypt(Box);

const { ciphertext } = yield* encrypt({ plaintext: bytes });
const { plaintext } = yield* decrypt({ ciphertext });
const raw = Redacted.value(plaintext);
```

```typescript
const sign = yield* Fly.Sign(Signing);
const verify = yield* Fly.Verify(Signing);

const { signature } = yield* sign({ plaintext: bytes });
yield* verify({ plaintext: bytes, signature });
```

Provide the matching `*Http` layer on the Service or Action.
Generate, set, and delete stay on the `SecretKey` resource.

Decrypt wraps plaintext in `Redacted`. Fly crypto ops need a KMS
token. Org API tokens are typed `Forbidden`. Encrypt and sign from
a Service, not a laptop Action.

## Where next

The [tutorial](../tutorial/part-4.md) reads a `Config.Redacted` secret
from a Service. [Services](../compute/services.md) covers
`Config.Redacted` and grouping Services in one App. [Postgres](postgres.md),
[Redis](redis.md), and [Tigris](tigris.md) bind
typed clients (`ConnectPostgres`, `ReadWriteRedis`, `PutObject`)
that use Fly-owned secrets internally. See the
[`Secret`](https://alchemy.run/providers/fly/reference/secret#secret),
[`GetSecret`](https://alchemy.run/providers/fly/reference/secret#getsecret),
[`ListSecrets`](https://alchemy.run/providers/fly/reference/secret#listsecrets),
[`SecretKey`](https://alchemy.run/providers/fly/reference/secret-key#secretkey),
[`Encrypt`](https://alchemy.run/providers/fly/reference/secret-key#encrypt),
[`Decrypt`](https://alchemy.run/providers/fly/reference/secret-key#decrypt),
[`Sign`](https://alchemy.run/providers/fly/reference/secret-key#sign), and
[`Verify`](https://alchemy.run/providers/fly/reference/secret-key#verify) references.
