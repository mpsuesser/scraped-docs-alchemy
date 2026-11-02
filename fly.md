---
url: https://alchemy.run/fly
title: "Fly"
description: "Deploy Effect programs to Fly.io as Apps, Machines, Services, and Sprites."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A Service is an Effect program running in Fly Machines inside its own App, with its own `{name}.fly.dev` hostname. An App is a globally unique namespace. A Machine is a Firecracker VM running a container. A Sprite is an Effect program in an org-scoped sandbox that hibernates when idle.

Disks, managed data, secrets, IPs, and certificates are declared in the same TypeScript program.

[Set up your org and API token](fly/setup.md), then start the [tutorial](fly/tutorial/part-1.md).

## Compute

- **[Apps](fly/compute/apps.md)** — the namespace for Machines, Secrets, IPs, and certificates. Services create their own; declare one for Machines, App-level Secrets, custom domains, or to group Services. `https://{appName}.fly.dev` is the public URL.
- **[Machines](fly/compute/machines.md)** — a Firecracker VM running a container image. Use this when you already have an image (`nginx:alpine`, a registry tag). Region, guest size, proxy services, env, and mounts are all props.
- **[Services](fly/compute/services.md)** — an Effect program running in Machines in its own App. Public by default at `api.url`, or private with `public: false`. Alchemy bundles `main`, builds the image, and updates in place when the hash changes. Set `count` to scale.
- **[Connect Services](fly/compute/connecting-services.md)** — Services calling each other’s methods with `Fly.bindService` on a stack-scoped private network (`Fly.stackNetwork`), behind one public gateway.
- **[Sprites](fly/compute/sprites.md)** — an Effect program in a Fly Sprite. Org-scoped, no parent App. Hibernates when idle. Alchemy writes the bundle onto the Sprite. No Docker image.
- **[Regions](fly/compute/regions.md)** — the datacenter a Machine, Volume, Postgres cluster, or Redis primary lives in. Default `iad`. Changing `region` replaces the resource.

## Frontend

- **[Websites](fly/frontend/websites.md)** — Vite, Astro, Next.js, Nuxt, React Router, SolidStart, SvelteKit, TanStack Start, Waku, Octane, Foldkit, Vocs, and StaticSite. Each builds your app and deploys a Node [Service](fly/compute/services.md) on a Fly App (`https://{app}.fly.dev`).
- [`Vite`](fly/frontend/vite.md), [`Astro`](fly/frontend/astro.md), [`Next.js`](fly/frontend/nextjs.md), [`Nuxt`](fly/frontend/nuxt.md), [`React Router`](fly/frontend/react-router.md), [`SolidStart`](fly/frontend/solidstart.md), [`SvelteKit`](fly/frontend/sveltekit.md), [`TanStack Start`](fly/frontend/tanstack-start.md), [`Waku`](fly/frontend/waku.md), [`Octane`](fly/frontend/octane.md), [`Foldkit`](fly/frontend/foldkit.md), [`Vocs`](fly/frontend/vocs.md), [`StaticSite`](fly/frontend/static-site.md).

## Data

- **[Volumes](fly/data/volumes.md)** — region-local block storage. Mount one into a Service with `MountVolume({ path, sizeGb })`, or pass `mounts` on a Machine. A Volume attaches to **one** Machine; `count: N` creates N disks, not one shared disk.
- **[Postgres](fly/data/postgres.md)** — billed Managed Postgres. Bind `ConnectPostgres` on a Service; pass the connection string to Drizzle or SQL. `migrations` is the same surface as Neon.
- **[Redis](fly/data/redis.md)** — Upstash Redis. Bind `ReadWriteRedis` on a Service.
- **[Tigris](fly/data/tigris.md)** — S3-compatible object storage (`Fly.Bucket`). Bind `PutObject` / `GetObject` on a Service.

## Secrets

- **[Secrets](fly/data/secrets.md)** — `Config.Redacted` in a Service for values from `.env`. `Fly.Secret` when Fly should own a value shared across Machines. KMS keys are `SecretKey` plus `Encrypt` / `Decrypt` / `Sign` / `Verify`.

## Networking

- **[IPs & certificates](fly/networking.md)** — public and private Services, addresses (`IpAssignment`) for Apps you declare, plus ACME or uploaded TLS certificates for your own hostname.

## What are you building?

| You’re building | Reach for |
| --- | --- |
| An HTTP API on Fly | [Service](fly/compute/services.md) |
| An internal API | [Service](fly/compute/services.md#private-services) with `public: false` |
| Several Services talking privately | [Connect Services](fly/compute/connecting-services.md): `Fly.bindService` + `Fly.stackNetwork` |
| A Vite SPA | [`Website.Vite`](fly/frontend/vite.md) |
| An SSR site (Astro, Next, Nuxt, …) | [`Website`](fly/frontend/websites.md) |
| A sandbox that can sleep | [Sprite](fly/compute/sprites.md) |
| A raw container | [Machine](fly/compute/machines.md) with an `image` |
| A background worker | [Service](fly/compute/services.md) with `ServerHost.run` |
| Persistent disk | [MountVolume](fly/data/volumes.md) or Machine `mounts` |
| Managed Postgres | [Postgres](fly/data/postgres.md) + `ConnectPostgres` |
| Upstash Redis | [Redis](fly/data/redis.md) + `ReadRedis` / `WriteRedis` / `ReadWriteRedis` |
| Object storage | [Tigris](fly/data/tigris.md) + `PutObject` / `GetObject` |
| N replicas behind fly.dev | Service `count`, or more [Machine](fly/compute/machines.md) resources |
| Restore a disk | [VolumeSnapshot](fly/data/volumes.md#snapshots) + mount `snapshotId` |
| Config / API tokens | [`Config.Redacted`](fly/data/secrets.md). [`Fly.Secret`](fly/data/secrets.md) when Fly should own it |
| Sign / encrypt in-app | [SecretKey](fly/data/secrets.md#kms-keys) + `Encrypt` / `Sign` |
| Addresses on your own App | [IpAssignment](fly/networking.md#addresses-for-a-shared-app) `type: "shared_v4"` |
| Your own domain | [Certificate](fly/networking.md#use-your-own-hostname) (ACME) |

## Where next

- [Setup](fly/setup.md) — create a Fly org, generate an API token, and connect alchemy.
- [Tutorial](fly/tutorial/part-1.md) — from empty directory to a public HTTP Service with a mounted Volume and a secret.
- [Websites](fly/frontend/websites.md) — Vite, Astro, Next.js, and other framework sites as a Node Service.
- [Providers reference](https://alchemy.run/providers) — generated API docs for every Fly resource Alchemy ships.
