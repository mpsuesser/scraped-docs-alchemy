---
url: https://alchemy.run/planetscale/data/credentials
title: "Credentials"
description: "PlanetScale credentials as Stack resources — Postgres roles, the default role and forceReset, MySQL passwords, TTLs, CIDR allowlists, and direct vs pooled connection origins."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

PlanetScale credentials are resources: Postgres branches use **roles** (`PostgresRole`, `PostgresDefaultRole`), MySQL branches use **passwords** (`MySQLPassword`). One rule shapes all three — PlanetScale returns the plaintext exactly once, at create time. Alchemy stores it `Redacted` in state, and any change it can’t apply in place mints a fresh credential whose new plaintext propagates to downstream resources automatically.

<svg viewBox="0 0 500 365" style="max-width: 575px" role="img" aria-label="PlanetScale never shows a password again, so Alchemy keeps it, redacted, and hands it on. A replaced credential reaches its consumers in the same deploy."><defs><marker id="alc-seq-arrow-kxw3db" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-kxw3db" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="361"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="42.5" text-anchor="middle">PlanetScale</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="361"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">Alchemy</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">state</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="361"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="36" text-anchor="middle">Hyperdrive</text> <text fill="currentColor" x="418" y="51" text-anchor="middle">or any consumer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="85" x2="88" y2="85" marker-end="url(#alc-seq-arrow-kxw3db)"></line><rect stroke="currentColor" fill="none" x="99.30000000000001" y="66" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="166" y="78" text-anchor="middle">create PostgresRole</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="125" x2="244" y2="125" marker-end="url(#alc-seq-arrow-kxw3db)"></line><rect stroke="currentColor" fill="none" x="72.9" y="106" width="186.2" height="15" rx="3"></rect><text fill="currentColor" x="166" y="118" text-anchor="middle">user + password, shown once</text></g> <g><rect stroke="currentColor" x="188.5" y="153" width="123" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="169" text-anchor="middle">stored Redacted</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="201" x2="412" y2="201" marker-end="url(#alc-seq-arrow-kxw3db)"></line><rect stroke="currentColor" fill="none" x="244.2" y="182" width="179.6" height="15" rx="3"></rect><text fill="currentColor" x="334" y="194" text-anchor="middle">password as a typed output</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="16" y1="249" x2="484" y2="249"></line><rect stroke="currentColor" fill="none" x="16" y="240" width="239.8" height="18" rx="4"></rect><text fill="currentColor" x="22" y="253">a change that can't apply in place</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="281" x2="88" y2="281" marker-end="url(#alc-seq-arrow-kxw3db)"></line><rect stroke="currentColor" fill="none" x="105.9" y="262" width="120.19999999999999" height="15" rx="3"></rect><text fill="currentColor" x="166" y="274" text-anchor="middle">create a new role</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="321" x2="412" y2="321" marker-end="url(#alc-seq-arrow-kxw3db)"></line><rect stroke="currentColor" fill="none" x="211.2" y="302" width="245.6" height="15" rx="3"></rect><text fill="currentColor" x="334" y="314" text-anchor="middle">the new password, on the same deploy</text></g></svg> <svg viewBox="0 0 322 364" style="max-width: 370px" role="img" aria-label="PlanetScale never shows a password again, so Alchemy keeps it, redacted, and hands it on. A replaced credential reaches its consumers in the same deploy."><defs><marker id="alc-seq-arrow-n-kxw3db" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-kxw3db" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="58" x2="52.333333333333336" y2="360"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="100.66666666666667" height="56" rx="8"></rect><text fill="currentColor" x="52.333333333333336" y="35" text-anchor="middle">PlanetScale</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="58" x2="161" y2="360"></line><rect stroke="currentColor" fill="none" x="110.66666666666666" y="2" width="100.66666666666667" height="56" rx="8"></rect><text fill="currentColor" x="161" y="22" text-anchor="middle">Alchemy</text> <text fill="currentColor" x="161" y="37" text-anchor="middle">state</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="269.66666666666663" y1="58" x2="269.66666666666663" y2="360"></line><rect stroke="currentColor" fill="none" x="219.3333333333333" y="2" width="100.66666666666667" height="56" rx="8"></rect><text fill="currentColor" x="269.66666666666663" y="22" text-anchor="middle">Hyperdrive</text> <text fill="currentColor" x="269.66666666666663" y="37" text-anchor="middle">or any</text> <text fill="currentColor" x="269.66666666666663" y="50" text-anchor="middle">consumer</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="84" x2="58.333333333333336" y2="84" marker-end="url(#alc-seq-arrow-n-kxw3db)"></line><rect stroke="currentColor" fill="none" x="39.966666666666676" y="65" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="106.66666666666667" y="77" text-anchor="middle">create PostgresRole</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="52.333333333333336" y1="124" x2="155" y2="124" marker-end="url(#alc-seq-arrow-n-kxw3db)"></line><rect stroke="currentColor" fill="none" x="13.566666666666677" y="105" width="186.2" height="15" rx="3"></rect><text fill="currentColor" x="106.66666666666667" y="117" text-anchor="middle">user + password, shown once</text></g> <g><rect stroke="currentColor" x="99.5" y="152" width="123" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="161" y="168" text-anchor="middle">stored Redacted</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="200" x2="263.66666666666663" y2="200" marker-end="url(#alc-seq-arrow-n-kxw3db)"></line><rect stroke="currentColor" fill="none" x="125.5333333333333" y="181" width="179.6" height="15" rx="3"></rect><text fill="currentColor" x="215.33333333333331" y="193" text-anchor="middle">password as a typed output</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="2" y1="248" x2="320" y2="248"></line><rect stroke="currentColor" fill="none" x="2" y="239" width="239.8" height="18" rx="4"></rect><text fill="currentColor" x="8" y="252">a change that can't apply in place</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="280" x2="58.333333333333336" y2="280" marker-end="url(#alc-seq-arrow-n-kxw3db)"></line><rect stroke="currentColor" fill="none" x="46.56666666666668" y="261" width="120.19999999999999" height="15" rx="3"></rect><text fill="currentColor" x="106.66666666666667" y="273" text-anchor="middle">create a new role</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="320" x2="263.66666666666663" y2="320" marker-end="url(#alc-seq-arrow-n-kxw3db)"></line><rect stroke="currentColor" fill="none" x="78.39999999999999" y="301" width="245.6" height="15" rx="3"></rect><text fill="currentColor" x="201.2" y="313" text-anchor="middle">the new password, on the same deploy</text></g></svg>

PlanetScale never shows a password again, so Alchemy keeps it, redacted, and hands it on. A replaced credential reaches its consumers in the same deploy.

## Postgres roles

`PostgresRole` mints a Postgres user + password on a branch; permissions come from `inheritedRoles`:

```typescript
import * as Planetscale from "@alchemy.run/planetscale";

const reader = yield* Planetscale.PostgresRole("reader", {
  database,
  branch, // defaults to "main"
  inheritedRoles: ["pg_read_all_data", "pg_read_all_settings"],
});
```

Only `name` and `successor` (the role that inherits ownership when this one is dropped) update in place — changing `ttl`, `inheritedRoles`, `withReplication`, `database`, or `branch` replaces the role with a new id, name, and password.

A logical-replication consumer (Electric, Debezium, a CDC pipeline) needs the `REPLICATION` attribute, which Postgres never grants through role membership. PlanetScale issues it only alongside `postgres`:

```typescript
const replicator = yield* Planetscale.PostgresRole("replicator", {
  database,
  inheritedRoles: ["postgres"],
  withReplication: true,
});
```

## The default role

Every Postgres branch ships with a built-in default role; `PostgresDefaultRole` captures it as a resource:

```typescript
const defaultRole = yield* Planetscale.PostgresDefaultRole("main-role", {
  database,
  forceReset: true,
});
```

The default role cannot be adopted: if it already exists and your Stack has no record of it, the deploy fails (`Default role already exists ... Use forceReset: true to reset`) rather than silently taking over. `forceReset: true` opts in by resetting it — same role, **new password** — which invalidates any credentials other systems were using. Destroying the resource also resets rather than deletes (PlanetScale has no delete endpoint for it). Prefer `PostgresRole` for application access.

## MySQL passwords

`MySQLPassword` mints credentials scoped to one branch and one role — `"reader"`, `"writer"`, `"admin"`, or `"readwriter"`:

```typescript
const password = yield* Planetscale.MySQLPassword("app-password", {
  database,
  branch, // defaults to "main"
  role: "reader",
});
```

`name` and `cidrs` update in place (same id, no new plaintext); changing `role`, `ttl`, `replica`, `database`, or `branch` replaces the password and deletes the old one.

## TTLs and CIDR allowlists

Both families take a `ttl` (seconds) for credentials that expire on their own; `MySQLPassword` additionally takes `cidrs` to restrict which IP ranges may connect:

```typescript
const admin = yield* Planetscale.MySQLPassword("admin-password", {
  database,
  role: "admin",
  ttl: 86400,
  cidrs: ["203.0.113.0/24"],
});
```

Changing `ttl` is a replacement on both roles and passwords (the expiry is baked in at create time); editing `cidrs` on a MySQL password syncs in place on the next deploy.

## origin vs pooledOrigin

Credentials expose parsed connection components ready to feed into consumers like [Cloudflare Hyperdrive](../../cloudflare/data/hyperdrive.md). `PostgresRole` has two: `origin` is the direct connection (port 5432), `pooledOrigin` goes through PSBouncer (port 6432):

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

const hyperdrive = yield* Cloudflare.Hyperdrive.Connection("app-hyperdrive", {
  origin: role.origin, // direct — Hyperdrive pools at the edge
  dev: role.pooledOrigin, // local dev bypasses Hyperdrive, so pool at PlanetScale
});
```

`MySQLPassword` exposes a single `origin`, and `PostgresDefaultRole` carries the same direct/pooled endpoints as `Redacted` URL strings (`connectionUrl` / `connectionUrlPooled`), as does `PostgresRole`.

## Where next

- [Postgres](postgres.md) — databases, branches, and roles in context.
- [MySQL](mysql.md) — the Vitess side, end to end.
- [Hyperdrive](../../cloudflare/data/hyperdrive.md) — edge-pooled connections from Workers to your PlanetScale database.

Reference:

- [PostgresRole](https://alchemy.run/providers/planetscale/postgres#postgresrole) · [PostgresDefaultRole](https://alchemy.run/providers/planetscale/postgres#postgresdefaultrole) · [MySQLPassword](https://alchemy.run/providers/planetscale/mysql#mysqlpassword)
