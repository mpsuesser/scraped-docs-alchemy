---
url: https://alchemy.run/turso/setup
title: "Setup"
description: "Give Alchemy a Turso API token."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

Sign up at [turso.tech](https://turso.tech/), then create an API token with the [Turso CLI](https://docs.turso.tech/cli/introduction):

```sh
turso auth api-tokens mint alchemy
```

Turso only supports API tokens. There is no browser sign-in.

## Add the provider

Install `@alchemy.run/turso`:

```sh
pnpm add @alchemy.run/turso@latest
```

Then add `Turso.providers()` next to your cloud’s providers:

```typescript
import * as Turso from "@alchemy.run/turso";

providers: Layer.mergeAll(Cloudflare.providers(), Turso.providers()),
```

## Store the token in a profile

```sh
alchemy profile edit --add Turso
```

Paste the token when asked. You can leave the organization blank.

If you belong to more than one organization, enter its name. `turso org list` shows them.

See [Profiles](../environments/profiles.md) to learn how profiles work.

## Use environment variables in CI

Set `TURSO_API_TOKEN` and Alchemy uses it instead of your profile:

```sh
TURSO_API_TOKEN=... alchemy deploy
```

`TURSO_API_KEY` works too. Set `TURSO_ORGANIZATION` if you belong to more than one organization.

## Pass a token in code

Use a token you already have, for example in tests:

```typescript
Effect.provide(
  Turso.fromToken({
    apiToken: "...",
    organization: "my-org",
  }),
)
```

`apiToken` can be a string or a `Redacted` value.

## Next steps

- [Build the tutorial](tutorial/part-1.md)
