---
url: https://alchemy.run/cloudflare/data/hyperdrive
title: "Hyperdrive"
description: "Cloudflare Hyperdrive pools connections to your external Postgres or MySQL database at the edge — provision a database (Neon or PlanetScale), front it with Hyperdrive, and bind the connection into your Worker."
access_date: 2026-08-03T19:00:17.443Z
current_date: 2026-08-03T19:00:17.443Z
---

Hyperdrive is a Cloudflare-managed connection pooler that sits between Workers and an external Postgres or MySQL database. The Worker sees a familiar connection string, but the connection itself is already pooled at the edge — no per-request TCP handshakes, no cold-start connection storms.

Reach for Hyperdrive whenever your app needs a real relational database: managed Postgres/MySQL providers like Neon and PlanetScale, or a database you already run. (For serverless SQLite inside Cloudflare, use [D1](d1.md) instead.) This page provisions a serverless database, fronts it with Hyperdrive, and binds the connection into your Worker.

Three providers are wired up out of the box. Pick whichever you prefer; the rest of the page follows your selection:

- **Neon** — serverless Postgres with copy-on-write branching.
- **PlanetScale (Postgres)** — managed Postgres with branch-per-PR workflows.
- **PlanetScale (MySQL)** — Vitess-backed MySQL, same branching model.

## Add the provider

Each database provider is its own `providers()` layer. Register it alongside `Cloudflare.providers()` in your stack:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "alchemy/Cloudflare";
import * as Neon from "alchemy/Neon";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyStack",
  {
    providers: Cloudflare.providers(),
    providers: Layer.mergeAll(Cloudflare.providers(), Neon.providers()),
    state: Alchemy.localState(),
  },
  // ...
);
```

Neon’s auth is API-key-based. The first `bun alchemy login` after this change adds a `Neon` step that either reads `NEON_API_KEY` from the environment (good for CI) or stores a key under `~/.alchemy/credentials/<profile>/neon-stored.json`.

## Provision the database

Create `src/Db.ts`. The shape is the same across providers — a top-level database resource, a branch, and (for PlanetScale) a credentials resource that owns the password.

A `Neon.Project` is the top-level container. It owns a default branch, a default role, and the WAL history that copy-on-write branches fork from:

```typescript
import * as Neon from "alchemy/Neon";
import * as Effect from "effect/Effect";

export const Db = Effect.gen(function* () {
  const project = yield* Neon.Project("app-db", {
    region: "aws-us-east-1",
  });

  const branch = yield* Neon.Branch("app-branch", {
    project,
  });

  return { project, branch };
});
```

`Neon.Branch` is a copy-on-write fork — cheap to create, fast to destroy, and ideal for preview environments. Each branch has its own connection string (`branch.connectionUri`) and a pooled variant (`branch.pooledConnectionUri`).

## Put Hyperdrive in front

Hyperdrive needs an `origin` describing where your database lives. Each provider exposes a pre-parsed `origin` output (`{ scheme, host, port, database, user, password }`) ready to feed straight in:

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Neon from "alchemy/Neon";
import * as Effect from "effect/Effect";

export const Db = Effect.gen(function* () {
  const project = yield* Neon.Project("app-db", { region: "aws-us-east-1" });
  const branch = yield* Neon.Branch("app-branch", { project });
  return { project, branch };
});

export const Hyperdrive = Effect.gen(function* () {
  const { branch } = yield* Db;
  return yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
    origin: branch.origin,
  });
});
```

`branch.origin` points at the **direct** (non-pooled) Neon endpoint — the recommended target when sitting behind Hyperdrive, since Hyperdrive already does its own connection pooling.

## Dev mode and the pooled origin

In `bun alchemy dev`, alchemy doesn’t create a Hyperdrive config at all — the local Worker gets a connection string that points straight at the database. With Hyperdrive out of the path, nothing pools those local connections, so point the `dev` override at the provider’s **own** pooled endpoint instead:

```typescript
export const Hyperdrive = Effect.gen(function* () {
  const { branch } = yield* Db;
  return yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
    origin: branch.origin,
    dev: branch.pooledOrigin,
  });
});
```

`branch.pooledOrigin` is the same connection parsed against Neon’s pgbouncer endpoint (the raw URI is `branch.pooledConnectionUri`). Deployed Workers still go through Hyperdrive at the direct endpoint; local dev goes through Neon’s pooler.

The pooled endpoint is also the right target when Hyperdrive isn’t in the picture at all. A Hyperdrive connection string only exists inside a deployed Worker — the binding resolves it at runtime — so anything else that talks to the database (CI jobs, a container, `psql` on your laptop) should connect to the pooled URI directly: `branch.pooledConnectionUri` on Neon, `role.connectionUrlPooled` on PlanetScale. Just don’t do the reverse — `origin` deliberately points Hyperdrive at the *direct* endpoint, because Hyperdrive already pools, and stacking it on top of the provider’s pooler means two poolers fighting over the same connections.

## Install the client library

The Worker uses a Node-compatible driver to talk to Hyperdrive over the binding. Pick the one that matches your engine:

[`pg`](https://node-postgres.com/) — node-postgres:

```sh
bun add pg
bun add -d @types/pg
```

## Bind Hyperdrive to your Worker

`Cloudflare.Hyperdrive.Connect(...)` returns a typed accessor for the runtime binding — connection string, host, port, user, password, database — plus a `raw` escape hatch for libraries that want the underlying `Hyperdrive` object:

```typescript
import * as Cloudflare from "alchemy/Cloudflare";
import * as Effect from "effect/Effect";
import * as Redacted from "effect/Redacted";
import * as HttpServerResponse from "effect/unstable/http/HttpServerResponse";
import { Client } from "pg";
import { Hyperdrive } from "./Db.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  {
    main: import.meta.url,
    compatibility: {
      // node-postgres needs Node.js APIs to run inside a Worker.
      flags: ["nodejs_compat"],
    },
  },
  Effect.gen(function* () {
    const hd = yield* Cloudflare.Hyperdrive.Connect(Hyperdrive);

    return {
      fetch: Effect.gen(function* () {
        const connectionString = Redacted.value(yield* hd.connectionString);
        const rows = yield* Effect.promise(async () => {
          // One client per request — Hyperdrive does the pooling.
          const client = new Client({ connectionString });
          await client.connect();
          try {
            const r = await client.query("SELECT now() as now");
            return r.rows;
          } finally {
            await client.end().catch(() => {});
          }
        });
        return yield* HttpServerResponse.json({ ok: true, rows });
      }),
    };
  }),
  }).pipe(Effect.provide(Cloudflare.Hyperdrive.ConnectBinding)),
) {}
```

Two things to notice (regardless of engine):

- The `nodejs_compat` flag is required because the driver uses Node’s `net` and `crypto` APIs.
- The fetch handler opens a fresh connection per request and ends it on the way out. This is intentional — Hyperdrive does the pooling on Cloudflare’s side, so the Worker doesn’t need its own. (The [Drizzle guide](drizzle.md) revisits this when we want long-lived clients.)

## Wire the resources into the stack

Update `alchemy.run.ts` to yield `Db` and `Hyperdrive` so they become part of the deploy graph:

```typescript
import Api from "./src/Api.ts";
import { Db, Hyperdrive } from "./src/Db.ts";

export default Alchemy.Stack(
  "MyStack",
  { /* ... */ },
  Effect.gen(function* () {
    yield* Db;
    yield* Hyperdrive;
    const api = yield* Api;
    return { url: api.url.as<string>() };
  }),
);
```

## Deploy

```sh
bun alchemy deploy
```

Watch the deploy plan: alchemy creates the database, then the branch, then the credentials (for PlanetScale), then the Hyperdrive that points at the connection origin, then the Worker with the Hyperdrive binding attached. After it completes, hit your Worker URL and you should see something like:

```json
{ "ok": true, "rows": [{ "now": "2026-05-04T07:53:48.123Z" }] }
```

You’re now talking to your database from the edge through Hyperdrive.

## Where next

Guides that build on Hyperdrive:

- [Add Drizzle ORM](drizzle.md) — swap the raw driver for typed schemas and alchemy-managed migrations.
- [Shared database across stages](shared-database.md) — point PR-preview stages at one long-lived database.
- [Branch from a shared database](branch-from-shared-database.md) — copy-on-write branches per preview stage.

Database integrations:

- [PlanetScale](../../planetscale.md)
- [Neon](../../neon.md)
