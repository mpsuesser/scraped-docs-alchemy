---
url: https://alchemy.run/prisma
title: "Prisma"
description: "Declare Prisma Postgres databases, connections, branches and Compute apps as Stack resources with Alchemy, and run the same Stack against a local Postgres in dev."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Prisma is best known for Prisma ORM, a TypeScript ORM that generates a typed client from your schema. Its data platform adds Prisma Postgres, a serverless database, and Prisma Compute, which runs the apps that use it.

**New to Prisma?** Start with [What is Prisma?](prisma/what-is-prisma.md).

## A quick example

A project with a database, and an app deployed next to it:

```typescript
const project = yield* Prisma.Project("app", { region: "us-east-1" });

const app = yield* Prisma.Compute("api", {
  project,
  path: "./app",
  build: "auto",
  healthCheck: { path: "/api/health" },
});
```

The app receives the database’s `DATABASE_URL` automatically. From any other runtime, bind a connection with `Prisma.Connect` and hand its URL to Prisma ORM, Effect SQL or Drizzle:

```typescript
const db = yield* Prisma.Connect(connection);
const sql = yield* SQL.Postgres({ url: db.databaseUrl });
```

## Where to start

**Want to build?** The [tutorial](prisma/tutorial/part-1.md) deploys a project, an Effect API, Postgres and a Vite frontend, one step at a time. Start with [Setup](prisma/setup.md) to connect your workspace.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Accounts & access

- [Workspaces and tokens](prisma/workspaces-and-tokens.md)

### Compute

- [Apps](prisma/compute/apps.md)
- [Deployments](prisma/compute/deployments.md)

### Guides

- [Connect from Workers](prisma/guides/cloudflare-workers.md)
