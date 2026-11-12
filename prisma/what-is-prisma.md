---
url: https://alchemy.run/prisma/what-is-prisma
title: "What is Prisma?"
description: "The concepts behind Prisma's data platform — workspaces, projects, Prisma Postgres, connections, branches, Compute apps, deployments and buckets — and how @alchemy.run/prisma models each of them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Prisma’s data platform runs two things for you: **Prisma Postgres**, a serverless Postgres database, and **Prisma Compute**, a runtime for the apps that use it. A **project** holds both, so an app and its database are deployed side by side, and the app finds its database without any configuration.

Take one small app: an API on Compute with a Postgres database, and a branch per preview. A workspace holds the project. Each branch of the project has its own database and app, and the app receives its database’s `DATABASE_URL`.

acmeworkspace

appproject · us-east-1

mainbranch · production

apiCompute app

dbPrisma Postgres

pr-42branch · preview

apiCompute app

dbPrisma Postgres

A workspace holds projects, and a service token deploys into one workspace. Each branch of a project groups a database and the apps that use it. The first branch is production, the rest are previews.

## Workspaces and tokens

A **workspace** holds your projects, members and bill. A **service token** belongs to a workspace, and Alchemy deploys into the workspace your token belongs to.

Connect it once, in [Setup](setup.md).

## Projects

A **project** groups a database and the apps that use it. By default, creating one creates a Prisma Postgres database too, in the region you choose:

```typescript
const project = yield* Prisma.Project("app", { region: "us-east-1" });
```

Apps in the project receive that database’s `DATABASE_URL` without any setup.

## Prisma Postgres

**Prisma Postgres** is serverless Postgres: there’s no instance to size, and you pay for what you use. Declare a database on its own when it needs a lifecycle separate from the project’s:

```typescript
const postgres = yield* Prisma.Postgres("db", { project });
```

See [Postgres](data/postgres.md).

## Connections

A **connection** is a set of credentials for one database. It gives you a ready-made `databaseUrl`, direct and pooled connection strings, and the parts of the URL separately, for poolers like Cloudflare Hyperdrive. Give each app its own.

```typescript
const connection = yield* Prisma.Connection("api", { database: postgres });
```

See [Connections](data/connections.md).

## Branches

A **branch** groups databases and apps under a git-style name. The project’s first branch has the **production** role. Every other branch is a **preview**, so apps on it get preview settings. Give each preview stage a branch, and its database and app are kept apart from production.

See [Branches](data/branches.md).

## Compute apps

**Prisma Compute** runs your app on Bun, next to its database. Point it at a directory, and Alchemy builds the app, uploads it, and starts it. Or define it inline, as an Effect program:

```typescript
const app = yield* Prisma.Compute("api", { project, path: "./app", build: "auto" });
```

The app listens on `PORT`, which Compute sets. There’s no Docker image and no registry.

See [Apps](compute/apps.md).

## Deployments

Each change to an app becomes a **deployment**. The new deployment starts next to the old one, must pass its **health check**, and only then takes traffic. If it fails after taking over, Alchemy restores the one before.

See [Deployments](compute/deployments.md).

## Buckets

A **bucket** stores files next to the database, through an S3-compatible API.

See [Buckets](data/buckets.md).

## Local development

Under `alchemy dev`, the same Stack runs against a local Postgres, started for you with nothing to create in the cloud, and each app runs its framework’s own dev server.

## How @alchemy.run/prisma models these concepts

Each of these concepts is something your Stack declares.

Your token picks the workspace, and `Prisma.providers()` deploys into it. A project, its databases, connections and branches are declarations. A Compute app is built and deployed from a directory or an Effect program. Inside any runtime, a Compute app, a Lambda Function or a Cloudflare Worker, `Prisma.Connect` turns a connection into a database URL for Prisma ORM, Effect SQL or Drizzle.

Next, build one: the [tutorial](tutorial/part-1.md) puts these ideas to work, one step at a time. Or read the concepts in depth, starting with [Postgres](data/postgres.md).
