---
url: https://alchemy.run/turso/databases
title: "Create & configure databases"
description: "Create, copy, and protect Turso databases."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A Turso database is a SQLite database that lives in a [group](groups.md).

## Create a database

```typescript
const group = yield* Turso.Group("Group", { location: "aws-us-east-1" });
const db = yield* Turso.Database("Db", { group: group.name });
```

`db.url` is the URL apps connect to. `db.hostname` and `db.httpUrl` are there too.

Moving a database to another group creates a new, empty one.

## Name it

```typescript
const db = yield* Turso.Database("Db", { group: group.name, name: "app" });
```

Without `name`, Alchemy picks one for each stage. Names must be unique in your organization.

## Apply migrations

```typescript
const db = yield* Turso.Database("Db", {
  group: group.name,
  migrations: "./migrations",
});
```

See [Migrations](migrations.md).

## Copy another database

```typescript
const preview = yield* Turso.Database("Preview", {
  group: group.name,
  seed: { type: "database", name: db.name },
});
```

The new database starts with all of `db`'s tables and rows. Add `timestamp` to copy it as it was at that time:

```typescript
seed: { type: "database", name: db.name, timestamp: "2026-10-07T12:00:00Z" },
```

`seed` only matters when the database is first created.

## Block reads or writes

```typescript
const db = yield* Turso.Database("Db", {
  group: group.name,
  blockWrites: true,
});
```

`blockWrites` makes the database read-only. `blockReads` turns it off completely. Remove the prop and deploy to undo it.

## Protect it from deletion

```typescript
const db = yield* Turso.Database("Db", {
  group: group.name,
  deleteProtection: true,
});
```

While this is on, you can't destroy the database. Set it to `false` and deploy first.

## Limit who can connect

```typescript
const db = yield* Turso.Database("Db", {
  group: group.name,
  allowedIps: ["203.0.113.0/24"],
});
```

`allowedIps` and `allowedAwsVpcIds` need a paid Turso plan.

## Encrypt with your own key

```typescript
const db = yield* Turso.Database("Db", {
  group: group.name,
  encryption: {
    key: Redacted.make(process.env.DB_KEY!),
    cipher: "aes256gcm",
  },
});
```

The key is 32 random bytes, base64-encoded. Changing `encryption` creates a new database. Encryption needs a Pro or Enterprise plan.

## Where next

- [Connections](connections.md)
- [Preview databases](preview-databases.md)
- [Database reference](https://alchemy.run/providers/turso/reference/database)
