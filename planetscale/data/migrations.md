---
url: https://alchemy.run/planetscale/data/migrations
title: "Migrations"
description: "Apply SQL migrations and seed data to PlanetScale databases and branches as part of every deploy — ordered, hashed, tracked, and run over short-lived credentials."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Every PlanetScale database and branch resource — `PostgresDatabase`, `PostgresBranch`, `MySQLDatabase`, `MySQLBranch` — accepts a a `migrations` prop: a folder of `.sql` files applied in order as part of every deploy. No separate migration step, no long-lived migration user — alchemy mints a short-lived credential for the run and deletes it afterwards.

<svg viewBox="0 0 736.8 124" style="max-width: 847px" role="img" aria-label="Each deploy mints a credential, applies only the files the branch hasn't run, and deletes the credential. No migration user outlives the deploy."><line stroke-opacity="0.2" stroke="currentColor" x1="220.8" y1="24" x2="220.8" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="612.8" y1="24" x2="612.8" y2="118"></line><g><text fill="currentColor" x="4" y="50">credential</text> <text fill="currentColor" x="4" y="64">minted for this deploy</text> <g><rect stroke="currentColor" fill="none" x="220.8" y="40" width="391.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="416.79999999999995" y="56" text-anchor="middle">exists only now</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="736.8" y2="74"></line><text fill="currentColor" x="4" y="94">pr-42</text> <text fill="currentColor" x="4" y="108">branch</text> <g><rect stroke="currentColor" fill="none" x="254.40000000000003" y="84" width="111.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="310.4" y="100" text-anchor="middle">history</text></g> <g><rect stroke="currentColor" fill="none" x="377.6" y="84" width="212.80000000000007" height="24" rx="5"></rect><text fill="currentColor" x="484.00000000000006" y="100" text-anchor="middle">apply 0003_refunds</text></g></g> <text fill="currentColor" x="220.8" y="18" text-anchor="middle">deploy starts</text> <text fill="currentColor" x="612.8" y="18" text-anchor="middle">deleted</text></svg> <svg viewBox="0 0 330 159" style="max-width: 379px" role="img" aria-label="Each deploy mints a credential, applies only the files the branch hasn't run, and deletes the credential. No migration user outlives the deploy."><line stroke-opacity="0.2" stroke="currentColor" x1="31.8" y1="24" x2="31.8" y2="153"></line><line stroke-opacity="0.2" stroke="currentColor" x1="254.4" y1="24" x2="254.4" y2="153"></line><g><text fill="currentColor" x="0" y="44">credential <tspan fill="currentColor" dx="8">minted for this deploy</tspan></text> <g><rect stroke="currentColor" fill="none" x="31.8" y="54" width="222.6" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="70" text-anchor="middle">exists only now</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">pr-42 <tspan fill="currentColor" dx="8">branch</tspan></text> <g><rect stroke="currentColor" fill="none" x="50.88" y="108" width="63.59999999999999" height="24" rx="5"></rect><text fill="currentColor" x="82.67999999999999" y="124" text-anchor="middle">history</text></g> <g><rect stroke="currentColor" fill="none" x="120.84" y="108" width="120.84" height="24" rx="5"></rect><text fill="currentColor" x="120.84" y="144" text-anchor="start">apply 0003_refunds</text></g></g> <text fill="currentColor" x="31.8" y="18" text-anchor="start">deploy starts</text> <text fill="currentColor" x="254.4" y="18" text-anchor="middle">deleted</text></svg>

Each deploy mints a credential, applies only the files the branch hasn't run, and deletes the credential. No migration user outlives the deploy.

## Database vs branch

On a database, migrations run against the default branch:

```typescript
const db = yield* Planetscale.PostgresDatabase("MyDb", {
  clusterSize: "PS_10",
  migrations: "./migrations/postgres",
});
```

On a branch, they run against that branch itself — the shape used in branch-per-PR workflows, where each preview stage forks a branch and migrates it independently:

```typescript
const branch = yield* Planetscale.PostgresBranch("app-branch", {
  database: db,
  migrations: "./migrations",
});
```

Pairing this with a `Drizzle.Schema` resource closes the loop: the schema resource regenerates pending migration SQL from your TypeScript schema, and the branch — wired to its `out` output — applies whatever is new, in one deploy:

```typescript
const schema = yield* Drizzle.Schema("app-schema", {
  schema: "./src/schema.ts",
  out: "./migrations",
});

const branch = yield* Planetscale.PostgresBranch("app-branch", {
  database,
  migrations: schema,
});
```

## Ordering and hashing

Files are discovered recursively under the migrations directory and sorted by their numeric prefix (`0001_init.sql`, `0002_users.sql`, … — Drizzle’s timestamp prefixes work too), falling back to name order for files without one. Each file’s contents are SHA-256 hashed and the hashes are persisted in the resource’s state (`migrationsHashes`).

Adding a new file is an in-place update: already-applied names are skipped and only the new file runs. Editing or deleting a file that has already been applied is rewritten history and cannot be replayed in place:

- A **non-production branch** is replaced (re-forked from its parent and migrated from scratch).
- A **current or desired production branch** fails the plan. Add a new forward migration instead of rewriting files that have already run.

Renaming an applied file without changing its contents is neither: its row moves to the new name and nothing is replaced. A new file must sort after every applied one, or the deploy fails with `NonLinearMigrationHistoryError`; rename it so it sorts last. See [Change migration history safely](../../sql/effect-sql/migrations.md#change-migration-history-safely).

## The tracking table

Applied migrations are recorded in Alchemy’s `__alchemy_migrations` table (created automatically), one row per file:

```sql
CREATE TABLE IF NOT EXISTS "__alchemy_migrations" (
  id SERIAL PRIMARY KEY,
  hash text NOT NULL,
  created_at bigint,
  name text,
  applied_at timestamp with time zone DEFAULT now()
);
```

(The MySQL flavor is the same shape with MySQL types.) On each deploy, files whose names already appear in the table are skipped **and** their content hashes must still match. A hash mismatch or a recorded name with no local file is a hard error, unless another file has the same contents, which is treated as a rename.

A database previously migrated with drizzle-kit or Prisma is [adopted automatically](../../sql/effect-sql/migrations.md#adopting-an-existing-database): its old tracking table’s history is copied in once and the old table is frozen. Use `migrations: { dir, table }` to put the bookkeeping in a differently-named table:

```typescript
const db = yield* Planetscale.PostgresDatabase("MyDb", {
  clusterSize: "PS_10",
  migrations: { dir: "./migrations/postgres", table: "my_migrations" },
});
```

## Engine differences

Both engines follow the same flow — mint a temporary credential, apply pending files, record them, delete the credential — but the details differ:

**Postgres** connects with a temporary role inheriting `postgres` (10-minute TTL, deleted after the run — the TTL bounds the orphan window if deletion hiccups). Each migration file runs inside a transaction together with its bookkeeping `INSERT`, so a failing statement rolls the whole file back and fails the deploy; migrations that already committed stay applied. Fix the file and re-deploy — the run resumes from the failed migration.

**MySQL (Vitess)** connects with a temporary `admin` password (same 10-minute TTL). Files are split on Drizzle’s `--> statement-breakpoint` marker — Vitess’s parser rejects the `-->` token, so alchemy strips the breakpoints and runs each statement individually. MySQL DDL commits implicitly, so a failing multi-statement migration can leave earlier statements applied; prefer one schema change per migration file.

## Seed data

`importFiles` lists additional `.sql` files to apply after migrations. Paths are resolved relative to the working directory:

```typescript
const db = yield* Planetscale.PostgresDatabase("MyDb", {
  clusterSize: "PS_10",
  migrations: "./migrations/postgres",
  importFiles: ["./seed/postgres.sql"],
});
```

Unlike migrations, import files are not recorded in the tracking table. Each file is content-hashed and re-applied whenever its contents change, so write them to be safe to run more than once (`INSERT ... ON CONFLICT DO NOTHING` and the like). Files whose hashes match the previous deploy are skipped.

Branches also support PlanetScale-native seeding at creation time: `seedData: "last_successful_backup"` restores the last successful backup’s schema and data into the new branch, and `backupId` restores a specific backup. Both are ignored if the branch already exists.

```typescript
const branch = yield* Planetscale.PostgresBranch("app-branch", {
  database: db,
  seedData: "last_successful_backup",
});
```

## Where next

Guides:

- [Add Drizzle ORM](../../cloudflare/data/drizzle.md) — typed schemas and generated migrations.
- [Branch from a shared database](../../cloudflare/data/branch-from-shared-database.md) — branch-per-PR preview environments.

Related:

- [Postgres](postgres.md) — databases, branches, and roles.
- [MySQL](mysql.md) — the Vitess-backed family.

Reference:

- [PostgresDatabase](https://alchemy.run/providers/planetscale/postgres#postgresdatabase) · [PostgresBranch](https://alchemy.run/providers/planetscale/postgres#postgresbranch)
- [MySQLDatabase](https://alchemy.run/providers/planetscale/mysql#mysqldatabase) · [MySQLBranch](https://alchemy.run/providers/planetscale/mysql#mysqlbranch)
