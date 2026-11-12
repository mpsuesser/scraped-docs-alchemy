---
url: https://alchemy.run/neon/data/migrations
title: "Migrations"
description: "Apply SQL migrations and seed data to Neon projects and branches as part of every deploy — ordered, hashed, and tracked in Alchemy's __alchemy_migrations table."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Both `Neon.Project` and `Neon.Branch` accept a `migrations` prop — a folder of `.sql` files (or a [`Drizzle.Schema`](../../sql/drizzle/migrations.md) resource) applied in order as part of every deploy. On a project, migrations run against the default branch’s primary database; on a branch, they run against the branch itself:

```typescript
const project = yield* Neon.Project("my-project", {
  migrations: "./migrations",
});
```

Because branches are copy-on-write forks, a branch created from a migrated parent already contains the parent’s schema — and its tracking table. Point the branch at the same `migrations` directory and only migrations added after the fork are applied:

```typescript
const featureBranch = yield* Neon.Branch("feature", {
  project,
  migrations: "./migrations",
});
```

<svg viewBox="0 0 644 128" style="max-width: 741px" role="img" aria-label="pr-42 forks after 0002, so it already has both migrations and their rows in the tracking table. Its deploy applies only 0003, and main never sees it until the file merges."><g><text fill="currentColor" x="4" y="30">main</text> <text fill="currentColor" x="4" y="44">production</text> <line stroke-opacity="0.2" stroke="currentColor" x1="88" y1="30" x2="628" y2="30" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="94">pr-42</text> <text fill="currentColor" x="4" y="108">preview</text> <path stroke="currentColor" fill="none" d="M 304 30 C 322 30, 312 94, 330 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="330" y1="94" x2="574" y2="94"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="569" y1="89" x2="579" y2="99"></line><line stroke-opacity="0.2" stroke="currentColor" x1="569" y1="99" x2="579" y2="89"></line></g></g><g><circle stroke="currentColor" fill="none" cx="142" cy="30" r="5.5"></circle><text fill="currentColor" x="142" y="19" text-anchor="middle">0001_init</text></g> <g><circle stroke="currentColor" fill="none" cx="228.4" cy="30" r="5.5"></circle><text fill="currentColor" x="228.4" y="19" text-anchor="middle">0002_orders</text></g> <g><circle stroke="currentColor" fill="none" cx="412" cy="94" r="5.5"></circle><text fill="currentColor" x="412" y="83" text-anchor="middle">0003_refunds</text></g></svg> <svg viewBox="0 0 328 158" style="max-width: 377px" role="img" aria-label="pr-42 forks after 0002, so it already has both migrations and their rows in the tracking table. Its deploy applies only 0003, and main never sees it until the file merges."><g><text fill="currentColor" x="0" y="16">main <tspan fill="currentColor" dx="8">production</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="155.60000000000002" y="94">pr-42 <tspan fill="currentColor" dx="8">preview</tspan></text> <path stroke="currentColor" fill="none" d="M 129.60000000000002 46 C 147.60000000000002 46, 137.60000000000002 124, 155.60000000000002 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="155.60000000000002" y1="124" x2="281.6" y2="124"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="276.6" y1="119" x2="286.6" y2="129"></line><line stroke-opacity="0.2" stroke="currentColor" x1="276.6" y1="129" x2="286.6" y2="119"></line></g></g><g><circle stroke="currentColor" fill="none" cx="38.400000000000006" cy="46" r="5.5"></circle><text fill="currentColor" x="9.600000000000005" y="35" text-anchor="start">0001_init</text></g> <g><circle stroke="currentColor" fill="none" cx="87.04" cy="46" r="5.5"></circle><text fill="currentColor" x="51.84" y="67" text-anchor="start">0002_orders</text></g> <g><circle stroke="currentColor" fill="none" cx="190.4" cy="124" r="5.5"></circle><text fill="currentColor" x="152" y="113" text-anchor="start">0003_refunds</text></g></svg>

pr-42 forks after 0002, so it already has both migrations and their rows in the tracking table. Its deploy applies only 0003, and main never sees it until the file merges.

## Ordering and hashing

Files are discovered recursively under the migrations directory and sorted by their numeric prefix (`0001_init.sql`, `0002_users.sql`, …), falling back to name order for files without one. Each file’s contents are SHA-256 hashed and the hashes are persisted in the resource’s state (`migrationsHashes`), so adding a migration file — or editing an existing one — is what marks the resource for an update on the next deploy. When nothing changed, the migration step is skipped entirely.

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

A database previously migrated with drizzle-kit or Prisma is [adopted automatically](../../sql/effect-sql/migrations.md#adopting-an-existing-database): its old tracking table’s history is copied in once and the old table is frozen. Projects deployed by older Alchemy versions (which used a `neon_migrations` table) are upgraded in place.

On each deploy, files whose names already appear in the table are skipped, and their content hashes must still match. Use `migrations: { dir, table }` to put the bookkeeping in a differently-named table:

```typescript
const project = yield* Neon.Project("my-project", {
  migrations: { dir: "./migrations", table: "my_migrations" },
});
```

## Transactional apply

Each pending migration runs inside a transaction together with its bookkeeping `INSERT` into the tracking table, so partial application is impossible: a failing statement rolls the whole file back and fails the deploy, while migrations that already committed stay applied. Fix the file and re-deploy — the run resumes from the failed migration.

## Seed data with importFiles

`importFiles` lists additional `.sql` files to apply after migrations. Paths are resolved relative to the working directory:

```typescript
const project = yield* Neon.Project("my-project", {
  migrations: "./migrations",
  importFiles: ["./seed/users.sql"],
});
```

Unlike migrations, import files are not recorded in the tracking table. Each file is content-hashed and re-applied whenever its contents change, so write them to be safe to run more than once (`INSERT ... ON CONFLICT DO NOTHING`, `CREATE OR REPLACE`, and the like). Files whose hashes match the previous deploy are skipped.

## Where next

Guides:

- [Add Drizzle ORM](../../cloudflare/data/drizzle.md) — typed schemas and generated migrations.

Related:

- [Neon overview](../../neon.md) — projects, branches, and composing with your cloud.
- [Setup](../setup.md) — credentials and provider registration.

Reference:

- [Project API reference](https://alchemy.run/providers/neon/reference/project#project)
- [Branch API reference](https://alchemy.run/providers/neon/reference/branch#branch)
