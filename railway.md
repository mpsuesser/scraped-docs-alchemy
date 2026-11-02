---
url: https://alchemy.run/railway
title: "Railway"
description: "Deploy Effect programs to Railway as Projects, Services, databases, Volumes, and Buckets."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A Project is a workspace-scoped namespace. A Service is a container — a public image, an Effect program Alchemy bundles, or a canvas Function. Postgres, MySQL, Mongo, Redis, Volumes, Variables, and Buckets live in the same TypeScript program.

The production environment is created with the Project. Extra environments (staging) are a separate resource. Workspace is **not** a resource — Alchemy uses `me.workspace ?? me.workspaces[0]`.

[Set up your workspace and API token](railway/setup.md), then start the [tutorial](railway/tutorial/part-1.md).

## Compute

- **[Projects](railway/compute/projects.md)** — the parent for every other resource. Alchemy generates a unique name unless you pass one. `https://railway.com/project/{projectId}` is the dashboard.
- **[Services](railway/compute/services.md)** — a container in a Project. Pass `image` for a public tag (`hashicorp/http-echo`), `main` for an Effect program, or `repo` for GitHub. `healthcheck` / `cronSchedule` / `buildCommand` match Railway IaC. `url` is `https://{name}.up.railway.app`. Railway load-balances public traffic; Alchemy does not pin a replica count.
- **[Functions, templates & VMs](railway/compute/functions.md)** — Effect-native canvas Functions (`main`, no registry), inline `source` / cron, marketplace templates, sandboxes, and cloud agents.
- **[Environments](railway/compute/environments.md)** — extra deploy environments under a Project. Do **not** recreate production — it is `project.environmentId`.
- **[Regions](railway/compute/regions.md)** — where a Service, Volume, or database instance runs (`us-west2`, `us-east4`, …). Buckets use a shorter set (`sjc`, `iad`, `ams`, `sin`).

## Frontend

- **[Websites](railway/frontend/websites.md)** — Vite, Astro, Next.js, Nuxt, React Router, SolidStart, SvelteKit, TanStack Start, Waku, Octane, Foldkit, Vocs, and StaticSite. Each builds your app and deploys a [Service](railway/compute/services.md) from a generated Dockerfile Railway builds. URL is `*.up.railway.app` or a [custom domain](railway/networking.md#custom-domains).
- [`Vite`](railway/frontend/vite.md), [`Astro`](railway/frontend/astro.md), [`Next.js`](railway/frontend/nextjs.md), [`Nuxt`](railway/frontend/nuxt.md), [`React Router`](railway/frontend/react-router.md), [`SolidStart`](railway/frontend/solidstart.md), [`SvelteKit`](railway/frontend/sveltekit.md), [`TanStack Start`](railway/frontend/tanstack-start.md), [`Waku`](railway/frontend/waku.md), [`Octane`](railway/frontend/octane.md), [`Foldkit`](railway/frontend/foldkit.md), [`Vocs`](railway/frontend/vocs.md), [`StaticSite`](railway/frontend/static-site.md).

## Data

- **[Volumes](railway/data/volumes.md)** — block disk in a Project. Create one, then mount it into a Service with `MountVolume(volume, { path })`. Snapshot with `VolumeBackup`.
- **[Postgres](railway/data/postgres.md)** — official SSL Postgres image, a Volume, `DATABASE_URL`, and an optional public TCP proxy. Bind `ConnectPostgres` on a Service; pass the connection string to Drizzle or SQL.
- **[MySQL](railway/data/mysql.md)** — official `mysql` image, `MYSQL_URL`, `ConnectMySQL`.
- **[Mongo](railway/data/mongo.md)** — official `mongo` image, `MONGO_URL`, `ConnectMongo`.
- **[Redis](railway/data/redis.md)** — `redis:7` or `bitnami/redis`. Bind `ReadWriteRedis` on a Service.
- **[Buckets](railway/data/buckets.md)** — S3-compatible object storage. Bind `PutObject` / `GetObject` on a Service.

## Variables

- **[Variables](railway/data/variables.md)** — `Config.Redacted` in a Service for values from `.env`. `Railway.Variable` when Railway should own a value shared across services. `Railway.ref(Db, "DATABASE_URL")` emits `${{Db.DATABASE_URL}}` (IaC `db.env.DATABASE_URL`). The plaintext is never stored in attributes.

## Networking

- **[Custom domains, TCP & private networks](railway/networking.md)** — your hostname on a Service (`CustomDomain`), public TCP for Postgres/MySQL/Mongo/Redis (`TcpProxy` on `*.proxy.rlwy.net`), and named private meshes (`PrivateNetwork` + `PrivateNetworkEndpoint`). HTTP Services already get a generated `*.up.railway.app` domain.

## What are you building?

| You’re building | Reach for |
| --- | --- |
| An HTTP API on Railway | [Project](railway/compute/projects.md) + [Service](railway/compute/services.md) |
| A Vite SPA | [`Website.Vite`](railway/frontend/vite.md) |
| An SSR site (Astro, Next, Nuxt, …) | [`Website`](railway/frontend/websites.md) |
| A raw container | [Service](railway/compute/services.md) with an `image` |
| An Effect Function (no registry) | [Function](railway/compute/functions.md) |
| A canvas / cron Function | [Function](railway/compute/functions.md) with `source` |
| A background worker | [Service](railway/compute/services.md) with `ServerHost.run` |
| Persistent disk | [Volume](railway/data/volumes.md) + `MountVolume` (one volume per service) |
| Volume snapshot | [VolumeBackup](railway/data/volumes.md#backups) |
| Postgres | [Postgres](railway/data/postgres.md) + `ConnectPostgres` |
| MySQL | [MySQL](railway/data/mysql.md) + `ConnectMySQL` |
| Mongo | [Mongo](railway/data/mongo.md) + `ConnectMongo` |
| Redis | [Redis](railway/data/redis.md) + `ReadRedis` / `WriteRedis` / `ReadWriteRedis` |
| Object storage | [Bucket](railway/data/buckets.md) + `PutObject` / `GetObject` |
| Staging next to production | [Environment](railway/compute/environments.md) |
| Config / API tokens | [`Config.Redacted`](railway/data/variables.md). [`Railway.Variable`](railway/data/variables.md) when Railway should own it |
| Cross-service env | [`Railway.ref`](railway/data/variables.md#variable-references) |
| Your own domain | [CustomDomain](railway/networking.md#custom-domains) |
| Laptop access to Postgres | [TcpProxy](railway/networking.md#tcp-proxies) (or Postgres `public: true`) |
| Custom private DNS | [PrivateNetwork](railway/networking.md#private-networks) |
| Trusted call between Services | [Schemaless RPC](railway/compute/services.md#schemaless-rpc) — `bindService` / `bindFunction`, private mesh only |

## Where next

- [Setup](railway/setup.md) — create a Railway workspace, generate an account token, and connect alchemy.
- [Tutorial](railway/tutorial/part-1.md) — from empty directory to a Project running an HTTP Service with a mounted Volume and a Variable.
- [Websites](railway/frontend/websites.md) — Vite, Astro, Next.js, and other framework sites as a container Service.
- `examples/railway-service` — every resource and binding in one stack (Project, Environment, Variable, Volume, Postgres, MySQL, Redis, Bucket, TcpProxy, image Service, Effect Function, canvas cron Function, Effect Api + Worker, `Railway.ref`, canvas Group).
- [Providers reference](https://alchemy.run/providers) — generated API docs for every Railway resource Alchemy ships.
