---
url: https://alchemy.run/cloudflare/data/prisma
title: "Prisma ORM with Postgres"
description: "Deploy Prisma ORM v8 on Cloudflare Workers with Neon Postgres, Hyperdrive, controlled migrations, and Effect-native queries."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

[Contracts](../../sql/prisma/contracts.md) · [Query API](../../sql/prisma/postgres.md) · [Runnable example](https://github.com/alchemy-run/alchemy/tree/main/examples/cloudflare-neon-prisma)

## Install the dependencies

```sh
pnpm add alchemy@latest @alchemy.run/cloudflare@latest @alchemy.run/neon@latest @alchemy.run/prisma@latest effect @prisma/orm-postgres arktype
pnpm add -D prisma
```

[Cloudflare setup](../setup.md) · [Neon setup](../../neon/setup.md) · [Define a contract](../../sql/prisma/contracts.md)

## Configure validation for Workers

```typescript
import { configure } from "arktype/config";

configure({ jitless: true });
```

Import the bootstrap before the contract builder:

```typescript
import "./validation.ts";
import { defineContract } from "@alchemy.run/prisma/ORM";
```

If your package uses `"sideEffects": false`, preserve this module:

```json
{ "sideEffects": ["./src/prisma/validation.ts"] }
```

PSL-first projects skip this step.

## Provision the database and migrations

```typescript
import * as Neon from "@alchemy.run/neon";
import * as Prisma from "@alchemy.run/prisma";
import * as Effect from "effect/Effect";

export const Db = Effect.gen(function* () {
  const contract = yield* Prisma.Contract("contract");
  const project = yield* Neon.Project("app-db");
  const branch = yield* Neon.Branch("app-branch", { project });

  yield* Prisma.Migrate("migrate", {
    url: branch.connectionUri,
    contract,
  });

  return branch;
});
```

[Migration workflow](../../sql/prisma/migrations.md)

## Add Hyperdrive

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Hyperdrive = Effect.gen(function* () {
  const branch = yield* Db;
  return yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
    origin: branch.origin,
    caching: { disabled: true },
  });
});
```

Caching is disabled so reads immediately reflect writes. [Hyperdrive guide](hyperdrive.md).

## Connect in a Worker

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as PrismaPostgres from "@alchemy.run/prisma/ORM/Postgres";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { contract } from "./prisma/contract.ts";
import { Hyperdrive } from "./db.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const conn = yield* Cloudflare.Hyperdrive.Connect(Hyperdrive);
    const db = yield* PrismaPostgres.Postgres(conn.connectionString, { contract });

    return {
      fetch: Effect.gen(function* () {
        const users = yield* db.orm.public.User.all();
        return yield* HttpServerResponse.json({ users });
      }),
    };
  }).pipe(Effect.provide(Cloudflare.Hyperdrive.ConnectBinding)),
) {}
```

[Connection lifecycle](../../sql/effect-sql/lifecycle.md)

## Use a PSL contract

```typescript
import { makeDatabase } from "./prisma/generated/client.ts";

const conn = yield* Cloudflare.Hyperdrive.Connect(Hyperdrive);
const db = yield* makeDatabase(conn.connectionString);
```

[Generate the PSL client](../../sql/prisma/contracts.md#query-a-psl-contract) before compiling. The handler and binding layer stay the same.

## Wire the stack

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Neon from "@alchemy.run/neon";
import * as Prisma from "@alchemy.run/prisma";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Api from "./src/api.ts";

export default Alchemy.Stack(
  "PrismaWorker",
  {
    providers: Layer.mergeAll(
      Cloudflare.providers(),
      Neon.providers(),
      Prisma.providers(),
    ),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.url };
  }),
);
```

## Deploy and query

```sh
pnpm alchemy deploy
curl https://<your-worker>.workers.dev
```

This demo route is public. Add authentication before exposing private data.

## Clean up

```sh
pnpm alchemy destroy
```

Destroy removes the example’s Worker, Hyperdrive configuration, and Neon resources, including the database data.

## Runnable examples

- [TypeScript-first example](https://github.com/alchemy-run/alchemy/tree/main/examples/cloudflare-neon-prisma).
- [PSL-first example](https://github.com/alchemy-run/alchemy/tree/main/examples/cloudflare-neon-prisma-psl).
- [Prisma queries and transactions](../../sql/prisma/postgres.md).
- [Choose another SQL provider](../../sql/databases.md).
