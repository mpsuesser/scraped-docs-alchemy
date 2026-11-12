---
url: https://alchemy.run/aws/data/rds
title: "RDS & Aurora"
description: "How RDS and Aurora run Postgres and MySQL inside your VPC, how the Aurora helper creates a cluster with its secret, subnets and instances, and how runtimes connect, migrate and scale it."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

RDS runs relational databases for you: Postgres, MySQL, MariaDB and others, with backups, patching and failover handled by AWS. **Aurora** is AWS’s own engine behind the Postgres and MySQL interfaces. It separates compute from storage, so a cluster’s instances share one copy of the data, replicated across three Availability Zones.

## A cluster is storage plus instances

An Aurora **cluster** owns the storage: one volume, grown automatically, kept in six copies across three zones. **Instances** are the compute that run queries against it. One instance is the **writer**. Others are **readers**, which serve reads from the same storage and take over if the writer fails.

<svg viewBox="0 0 600 298" style="max-width: 690px" role="img" aria-label="The instances in each zone share one volume. It keeps two copies in every zone, so losing a zone loses no data. If the writer fails, a reader in another zone takes over."><defs><marker id="alc-z-arrow-91wkvy" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><rect stroke="currentColor" fill="none" x="8" y="8" width="584" height="282" rx="12"></rect><text fill="currentColor" x="22" y="28">Aurora cluster</text> <g><rect stroke="currentColor" fill="none" x="22" y="46" width="176" height="230" rx="8"></rect><text fill="currentColor" x="110" y="64" text-anchor="middle">us-east-1a</text></g> <g><rect stroke="currentColor" fill="none" x="212" y="46" width="176" height="230" rx="8"></rect><text fill="currentColor" x="300" y="64" text-anchor="middle">us-east-1b</text></g> <g><rect stroke="currentColor" fill="none" x="402" y="46" width="176" height="230" rx="8"></rect><text fill="currentColor" x="490" y="64" text-anchor="middle">us-east-1c</text></g> <rect stroke="currentColor" fill="none" x="30" y="168" width="540" height="96" rx="9"></rect><text fill="currentColor" x="300" y="190" text-anchor="middle">cluster volume</text> <text fill="currentColor" x="300" y="206" text-anchor="middle">one copy of the data, grown automatically</text> <g><rect stroke="currentColor" fill="none" x="54" y="222" width="52" height="26" rx="4"></rect><text fill="currentColor" x="80" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="114" y="222" width="52" height="26" rx="4"></rect><text fill="currentColor" x="140" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="244" y="222" width="52" height="26" rx="4"></rect><text fill="currentColor" x="270" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="304" y="222" width="52" height="26" rx="4"></rect><text fill="currentColor" x="330" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="434" y="222" width="52" height="26" rx="4"></rect><text fill="currentColor" x="460" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="494" y="222" width="52" height="26" rx="4"></rect><text fill="currentColor" x="520" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="38" y="76" width="144" height="46" rx="8"></rect><text fill="currentColor" x="110" y="96" text-anchor="middle">writer</text> <text fill="currentColor" x="110" y="112" text-anchor="middle">instance</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="110" y1="124" x2="110" y2="166" marker-end="url(#alc-z-arrow-91wkvy)"></line><text fill="currentColor" x="118" y="149">writes</text></g></g> <g><rect stroke="currentColor" fill="none" x="228" y="76" width="144" height="46" rx="8"></rect><text fill="currentColor" x="300" y="96" text-anchor="middle">reader</text> <text fill="currentColor" x="300" y="112" text-anchor="middle">instance</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="300" y1="166" x2="300" y2="124" marker-end="url(#alc-z-arrow-91wkvy)"></line><text fill="currentColor" x="308" y="149">reads</text></g></g> <g><rect stroke="currentColor" fill="none" x="418" y="76" width="144" height="46" rx="8"></rect><text fill="currentColor" x="490" y="96" text-anchor="middle">reader</text> <text fill="currentColor" x="490" y="112" text-anchor="middle">instance</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="490" y1="166" x2="490" y2="124" marker-end="url(#alc-z-arrow-91wkvy)"></line><text fill="currentColor" x="498" y="149">reads</text></g></g></svg> <svg viewBox="0 0 360 298" style="max-width: 414px" role="img" aria-label="The instances in each zone share one volume. It keeps two copies in every zone, so losing a zone loses no data. If the writer fails, a reader in another zone takes over."><defs><marker id="alc-z-arrow-n-91wkvy" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><rect stroke="currentColor" fill="none" x="8" y="8" width="344" height="282" rx="12"></rect><text fill="currentColor" x="22" y="28">Aurora cluster</text> <g><rect stroke="currentColor" fill="none" x="22" y="46" width="100" height="230" rx="8"></rect><text fill="currentColor" x="72" y="64" text-anchor="middle">us-east-1a</text></g> <g><rect stroke="currentColor" fill="none" x="130" y="46" width="100" height="230" rx="8"></rect><text fill="currentColor" x="180" y="64" text-anchor="middle">us-east-1b</text></g> <g><rect stroke="currentColor" fill="none" x="238" y="46" width="100" height="230" rx="8"></rect><text fill="currentColor" x="288" y="64" text-anchor="middle">us-east-1c</text></g> <rect stroke="currentColor" fill="none" x="30" y="168" width="300" height="96" rx="9"></rect><text fill="currentColor" x="180" y="190" text-anchor="middle">cluster volume</text> <text fill="currentColor" x="180" y="206" text-anchor="middle">one copy of the data, grown automatically</text> <g><rect stroke="currentColor" fill="none" x="34" y="222" width="34" height="26" rx="4"></rect><text fill="currentColor" x="51" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="76" y="222" width="34" height="26" rx="4"></rect><text fill="currentColor" x="93" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="142" y="222" width="34" height="26" rx="4"></rect><text fill="currentColor" x="159" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="184" y="222" width="34" height="26" rx="4"></rect><text fill="currentColor" x="201" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="250" y="222" width="34" height="26" rx="4"></rect><text fill="currentColor" x="267" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="292" y="222" width="34" height="26" rx="4"></rect><text fill="currentColor" x="309" y="239" text-anchor="middle">copy</text></g> <g><rect stroke="currentColor" fill="none" x="30" y="76" width="84" height="46" rx="8"></rect><text fill="currentColor" x="72" y="96" text-anchor="middle">writer</text> <text fill="currentColor" x="72" y="112" text-anchor="middle">instance</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="72" y1="124" x2="72" y2="166" marker-end="url(#alc-z-arrow-n-91wkvy)"></line><text fill="currentColor" x="80" y="149">writes</text></g></g> <g><rect stroke="currentColor" fill="none" x="138" y="76" width="84" height="46" rx="8"></rect><text fill="currentColor" x="180" y="96" text-anchor="middle">reader</text> <text fill="currentColor" x="180" y="112" text-anchor="middle">instance</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="180" y1="166" x2="180" y2="124" marker-end="url(#alc-z-arrow-n-91wkvy)"></line><text fill="currentColor" x="188" y="149">reads</text></g></g> <g><rect stroke="currentColor" fill="none" x="246" y="76" width="84" height="46" rx="8"></rect><text fill="currentColor" x="288" y="96" text-anchor="middle">reader</text> <text fill="currentColor" x="288" y="112" text-anchor="middle">instance</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="288" y1="166" x2="288" y2="124" marker-end="url(#alc-z-arrow-n-91wkvy)"></line><text fill="currentColor" x="296" y="149">reads</text></g></g></svg>

The instances in each zone share one volume. It keeps two copies in every zone, so losing a zone loses no data. If the writer fails, a reader in another zone takes over.

Instances come in two kinds. **Serverless v2** instances scale their capacity, measured in ACUs, up and down with load. **Provisioned** instances have a fixed size, like `db.r6g.large`.

## It lives in your VPC

A database has no public address. Its instances sit in private subnets of a VPC, and a **security group** decides who may connect on the database’s port. Its admin password is kept in **Secrets Manager**, never in code.

So a database starts with a network. The `Network` helper builds one, and a security group lets one other group in on port 5432:

```typescript
const network = yield* AWS.EC2.Network("Network", { cidrBlock: "10.0.0.0/16", nat: "single" });

const appSecurityGroup = yield* AWS.EC2.SecurityGroup("App", { vpcId: network.vpcId });

const databaseSecurityGroup = yield* AWS.EC2.SecurityGroup("Database", {
  vpcId: network.vpcId,
  ingress: [
    { ipProtocol: "tcp", fromPort: 5432, toPort: 5432, referencedGroupId: appSecurityGroup.groupId },
  ],
});
```

Anything in `appSecurityGroup` can reach the database. Nothing else can. See [VPC & networking](../networking.md).

## Create a cluster

`AWS.RDS.Aurora` creates everything a cluster needs from the subnets and security groups it lives in:

```typescript
const db = yield* AWS.RDS.Aurora("Database", {
  subnetIds: network.privateSubnetIds,
  securityGroupIds: [databaseSecurityGroup.groupId],
});
```

That’s a Secrets Manager secret with a generated password, a subnet group, an Aurora PostgreSQL cluster with a database named `app` and the Data API turned on, and one Serverless v2 writer scaling between 0.5 and 1 ACU. Set `engine: "aurora-mysql"` for MySQL.

Each part is on the result: `db.secret`, `db.cluster`, `db.writer`, `db.readers`. Configure any of them through the helper’s props, or declare the `AWS.RDS.DB*` resources yourself when you need full control.

## Apply migrations on deploy

Point `migrations` at a directory of `.sql` files. Each deploy applies the new ones once the writer is ready, in order, and records them in an `__alchemy_migrations` table:

```typescript
const db = yield* AWS.RDS.Aurora("Database", {
  subnetIds: network.privateSubnetIds,
  securityGroupIds: [databaseSecurityGroup.groupId],
  migrations: "./migrations",
});
```

Migrations travel over the Data API, so the machine running the deploy needs no route into the VPC. Over the Data API each statement is its own call: put one statement per file, or separate them with `--> statement-breakpoint` as drizzle-kit does. A database already migrated by drizzle-kit or Prisma is picked up where it left off.

Without the Data API, declare an `AWS.RDS.MigrationRunner`, a Function inside the VPC that applies them, and pass it as `migrationsTransport`. See [SQL migrations](../../sql/effect-sql/migrations.md).

## Connect with a driver

A runtime inside the VPC connects with an ordinary Postgres driver. Bind `AWS.RDS.Connect` to the cluster, and it resolves the host, port and credentials from the secret each time you ask, granting the runtime permission to read that secret:

```typescript
const connect = yield* AWS.RDS.Connect(db.cluster, { secret: db.secret, database: "app" });

const sql = yield* SQL.Postgres({
  url: connect.pipe(Effect.map((connection) => connection.url)),
});
```

The password never lands in an environment variable, and a rotated secret is picked up on the next connection. `SQL.Postgres` opens its pool on the first query of an invocation and closes it when the invocation ends. See [Effect SQL on Postgres](../../sql/effect-sql/postgres.md).

A Lambda Function joins the VPC with its `vpc` prop:

```typescript
{ main: import.meta.url, vpc: { subnetIds: network.privateSubnetIds, securityGroupIds: [appSecurityGroup.groupId] } }
```

## Query without a connection

The **Data API** runs SQL over HTTPS instead. There’s no driver, no pool and no VPC to join, so any runtime can use it. Bind `AWS.RDSData.ExecuteStatement`:

```typescript
const execute = yield* AWS.RDSData.ExecuteStatement(db.cluster, { secret: db.secret, database: "app" });
const result = yield* execute({ sql: "select now()" });
```

The binding grants `rds-data:ExecuteStatement` on the cluster and read access to the secret. Provide `AWS.RDSData.ExecuteStatementHttp` to the runtime. Transactions use `BeginTransaction`, `CommitTransaction` and `RollbackTransaction`, each a binding of its own.

Each Data API call adds a little latency over a warm connection. It suits functions that run a few queries per invocation, where opening a connection would cost more.

## Add readers, a proxy and capacity

Grow the same declaration as load grows:

```typescript
const db = yield* AWS.RDS.Aurora("Database", {
  subnetIds: network.privateSubnetIds,
  securityGroupIds: [databaseSecurityGroup.groupId],
  readers: 2,
  scaling: { minCapacity: 0.5, maxCapacity: 8 },
  proxy: true,
});
```

`readers` adds instances that serve reads and take over on failure. `scaling` sets the Serverless v2 range. Use `instanceClass` for a fixed instance size instead.

`proxy` adds an **RDS Proxy** in front of the cluster. Lambda can start hundreds of environments at once, each opening its own connections, and a database only accepts so many. The proxy holds a shared pool and hands connections out, so a traffic spike doesn’t exhaust the database. Bind `AWS.RDS.Connect` to `db.proxy` instead of the cluster to use it.

## Keep the data across deploys

Destroying the cluster deletes its data, with no final snapshot. Protect a production database from an accidental destroy with `deletionProtection`, and keep backups for longer than the default day:

```typescript
const db = yield* AWS.RDS.Aurora("Database", {
  subnetIds: network.privateSubnetIds,
  securityGroupIds: [databaseSecurityGroup.groupId],
  deletionProtection: true,
  backupRetentionPeriod: "14 days",
});
```

AWS then refuses to delete the cluster until you turn the protection off. Within the retention period, you can restore the cluster to any point in time.

For the whole app, from network to Drizzle queries, see [Drizzle + Aurora](drizzle-aurora.md). For a serverless database with no VPC at all, see [Drizzle + DSQL](drizzle-dsql.md).
