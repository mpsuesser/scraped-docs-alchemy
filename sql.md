---
url: https://alchemy.run/sql
title: "SQL"
description: "One home for SQL in alchemy — low-level effect-sql clients, Drizzle ORM, schema migrations in the deploy graph, and the per-execution connection lifecycle."
access_date: 2026-08-03T19:08:21.153Z
current_date: 2026-08-03T19:08:21.153Z
---

An alchemy app talks SQL at whichever level fits: a raw tagged-template client, an ORM, or both against the same connection. The schema rides the same deploy graph as the infrastructure, so `alchemy deploy` regenerates and applies pending migrations alongside everything else.

## Databases

- [D1](cloudflare/data/d1.md) — Cloudflare’s serverless SQLite, bound natively into a Worker.
- [Hyperdrive](cloudflare/data/hyperdrive.md) — edge connection pooling for any Postgres: [Neon](neon.md), [PlanetScale](planetscale.md), RDS, or a database you already run.
- AWS-side connections — DSQL, RDS, and Redshift bindings in the [API reference](https://alchemy.run/providers).

## Clients

`alchemy/SQL/*` is the low-level home: tagged-template queries with typed errors, no ORM — one subpath per backend (`alchemy/SQL/D1`, `alchemy/SQL/Postgres`), so you only load the driver you use. `alchemy/Drizzle` is the ORM sibling: a typed schema and relational queries. Both wrap the same [`@effect/sql`](https://effect.website/) drivers and share one lifecycle.

```typescript
import * as SQL from "alchemy/SQL/D1";
import * as Drizzle from "alchemy/Drizzle";

// Raw effect-sql — tagged-template queries, typed errors
const sql = yield* SQL.D1(d1);
const users = yield* sql\`SELECT * FROM users WHERE id = ${id}\`;

// Drizzle — typed schema, relational queries
const db = yield* Drizzle.D1(d1, { relations });
const user = yield* db.query.Users.findFirst({ with: { posts: true } });
```

Every client builds lazily on the first query of an execution, is reused for every query in that execution, and tears down when the event settles — [Connection lifecycle](sql/effect-sql/lifecycle.md) explains why.

## What are you building?

| Goal | Reach for |
| --- | --- |
| Raw SQL on Postgres | [Effect SQL: Postgres](sql/effect-sql/postgres.md) |
| Raw SQL on D1 | [Effect SQL: D1](sql/effect-sql/d1.md) |
| Typed schema + queries on Postgres | [Drizzle: Postgres](sql/drizzle/postgres.md) |
| Typed schema + queries on D1 | [Drizzle: D1](sql/drizzle/d1.md) |
| Schema changes applied on deploy | [Drizzle migrations](sql/drizzle/migrations.md) |
| Hand-written `.sql` migrations | [Effect SQL migrations](sql/effect-sql/migrations.md) |
| A service that runs on any database | `SqlClient` + Layers — see [Provide as a service](sql/effect-sql/postgres.md#provide-as-a-service) |

MySQL support is in flight: drizzle’s `effect-mysql2` driver and Hyperdrive’s text-protocol requirements are being reconciled upstream in `@effect/sql-mysql2`. Postgres and D1 are the supported paths today.

## Where next

- [Effect SQL: Postgres](sql/effect-sql/postgres.md) / [D1](sql/effect-sql/d1.md) — the raw clients.
- [Drizzle: Postgres](sql/drizzle/postgres.md) / [D1](sql/drizzle/d1.md) — schema to queries, end to end.
- [Add Drizzle ORM](cloudflare/data/drizzle.md) — full Worker wiring on Postgres via Hyperdrive.
- [Drizzle on D1](cloudflare/data/d1-drizzle.md) — full Worker wiring on D1.
- [API reference](https://alchemy.run/providers) — every SQL and Drizzle export.
