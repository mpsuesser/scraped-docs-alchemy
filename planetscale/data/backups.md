---
url: https://alchemy.run/planetscale/data/backups
title: "Backups & restores"
description: "Restore a PlanetScale backup into a fresh branch with backupId, or seed a new branch from the last successful backup with seedData."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Alchemy doesn’t manage PlanetScale backup schedules — create backups in the PlanetScale dashboard or via the API. What it gives you is the restore side: point a new branch at a backup and it comes up with that backup’s schema and data.

Both `MySQLBranch` and `PostgresBranch` accept the same two props: `backupId` restores a specific backup, `seedData` restores the parent branch’s last successful backup.

<svg viewBox="0 0 708 128" style="max-width: 814px" role="img" aria-label="A restored branch starts from the backup, not from main as it is now, so it comes up without the changes made after the backup."><g><text fill="currentColor" x="4" y="30">main</text> <text fill="currentColor" x="4" y="44">production</text> <line stroke-opacity="0.2" stroke="currentColor" x1="152" y1="30" x2="692" y2="30" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="94">restored</text> <text fill="currentColor" x="4" y="108">backupId: backup-123</text> <path stroke="currentColor" fill="none" d="M 314 30 C 332 30, 322 94, 340 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="340" y1="94" x2="692" y2="94"></line></g><g><circle stroke="currentColor" fill="none" cx="314" cy="30" r="5.5"></circle><text fill="currentColor" x="314" y="19" text-anchor="middle">backup-123</text></g> <g><circle stroke="currentColor" fill="none" cx="503" cy="30" r="5.5"></circle><text fill="currentColor" x="503" y="19" text-anchor="middle">bad migration</text></g> <g><circle stroke="currentColor" fill="none" cx="530" cy="94" r="5.5"></circle><text fill="currentColor" x="530" y="115" text-anchor="middle">schema + data as of the backup</text></g></svg> <svg viewBox="0 0 328 158" style="max-width: 377px" role="img" aria-label="A restored branch starts from the backup, not from main as it is now, so it comes up without the changes made after the backup."><g><text fill="currentColor" x="0" y="16">main <tspan fill="currentColor" dx="8">production</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="123.19999999999999" y="94">restored <tspan fill="currentColor" dx="8">backupId: backup-123</tspan></text> <path stroke="currentColor" fill="none" d="M 99.2 46 C 117.2 46, 107.2 124, 125.2 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="125.2" y1="124" x2="312" y2="124"></line></g><g><circle stroke="currentColor" fill="none" cx="99.2" cy="46" r="5.5"></circle><text fill="currentColor" x="67.2" y="35" text-anchor="start">backup-123</text></g> <g><circle stroke="currentColor" fill="none" cx="205.6" cy="46" r="5.5"></circle><text fill="currentColor" x="164" y="35" text-anchor="start">bad migration</text></g> <g><circle stroke="currentColor" fill="none" cx="220.79999999999998" cy="124" r="5.5"></circle><text fill="currentColor" x="124.79999999999998" y="145" text-anchor="start">schema + data as of the backup</text></g></svg>

A restored branch starts from the backup, not from main as it is now, so it comes up without the changes made after the backup.

## Restore a branch from a backup

Pass the backup’s ID as `backupId`. `clusterSize` is required when restoring from a backup:

```typescript
const restored = yield* Planetscale.MySQLBranch("restored", {
  database,
  parentBranch: "main",
  isProduction: true,
  backupId: "backup-123",
  clusterSize: "PS_10",
});
```

The branch is created from `parentBranch` with the backup’s schema and data restored into it, then waits until it’s ready like any other branch. Grab the ID from the backup’s page in the PlanetScale dashboard or the backups API, and make sure the backup has completed (state `success`) before deploying.

## Seed from the last successful backup

When you don’t care which backup — just “recent production data” — use `seedData` instead of pinning an ID:

```typescript
const preview = yield* Planetscale.MySQLBranch("preview", {
  database,
  parentBranch: "main",
  isProduction: false,
  seedData: "last_successful_backup",
});
```

This is the usual choice for preview and development branches that want realistic data without tracking backup IDs in code.

## Create-only semantics

Both props apply only when the branch is first created. If the branch already exists, `backupId` and `seedData` are ignored — subsequent deploys sync the branch’s other settings (production status, `clusterSize`, `safeMigrations`) but never re-restore data. Changing `backupId` on a deployed branch is a no-op, not a replacement.

To restore again, create a new branch: give the resource a new logical ID (or a new `name`) so the next deploy creates a fresh branch from the backup.

## Where next

- [MySQL](mysql.md) — databases, branches, and passwords.
- [Postgres](postgres.md) — the PostgreSQL side, including roles.
- [Migrations](migrations.md) — schema migrations and seed files per branch.

Reference:

- [MySQLBranch](https://alchemy.run/providers/planetscale/mysql#mysqlbranch) · [PostgresBranch](https://alchemy.run/providers/planetscale/postgres#postgresbranch)
