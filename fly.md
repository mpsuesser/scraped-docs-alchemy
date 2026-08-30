---
url: https://alchemy.run/fly
title: "Fly"
description: "Deploy Effect programs to Fly.io as Apps, Machines, Services, and Sprites."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

An App is a globally unique namespace. A Machine is a Firecracker VM running a container. A Service is an Effect program running in a Fly Machine. A Sprite is an Effect program in an org-scoped sandbox that hibernates when idle.

Disks, managed data, secrets, IPs, and certificates are declared in the same TypeScript program.

[Set up your org and API token](fly/setup.md), then start the [tutorial](https://alchemy.run/fly/tutorial/part-1).

## Compute

- **[Apps](https://alchemy.run/fly/compute/apps)** — the parent for every other resource except Sprites, Postgres, Redis, and Tigris. Alchemy generates a globally unique name unless you pass one. `https://{appName}.fly.dev` is the public URL.
- **[Machines](https://alchemy.run/fly/compute/machines)** — a Firecracker VM running a container image. Use this when you already have an image (`nginx:alpine`, a registry tag). Region, guest size, proxy services, env, and mounts are all props.
- **[Services](https://alchemy.run/fly/compute/services)** — an Effect program running in a Machine. Alchemy bundles `main`, builds the image, and updates in place when the hash changes. Set `count` to scale.
- **[Sprites](https://alchemy.run/fly/compute/sprites)** — an Effect program in a Fly Sprite. Org-scoped, no parent App. Hibernates when idle. Alchemy writes the bundle onto the Sprite. No Docker image.
- **[Regions](https://alchemy.run/fly/compute/regions)** — the datacenter a Machine, Volume, Postgres cluster, or Redis primary lives in. Default `iad`. Changing `region` replaces the resource.

## Frontend

- **[Websites](fly/frontend/websites.md)** — Vite, Astro, Next.js, Nuxt, React Router, SolidStart, SvelteKit, TanStack Start, Waku, Octane, Foldkit, Vocs, and StaticSite. Each builds your app and deploys a Node [Service](https://alchemy.run/fly/compute/services) on a Fly App (`https://{app}.fly.dev`).
- [`Vite`](fly/frontend/vite.md), [`Astro`](fly/frontend/astro.md), [`Next.js`](fly/frontend/nextjs.md), [`Nuxt`](fly/frontend/nuxt.md), [`React Router`](fly/frontend/react-router.md), [`SolidStart`](fly/frontend/solidstart.md), [`SvelteKit`](fly/frontend/sveltekit.md), [`TanStack Start`](fly/frontend/tanstack-start.md), [`Waku`](fly/frontend/waku.md), [`Octane`](fly/frontend/octane.md), [`Foldkit`](fly/frontend/foldkit.md), [`Vocs`](fly/frontend/vocs.md), [`StaticSite`](fly/frontend/static-site.md).

## Data

- **[Volumes](https://alchemy.run/fly/data/volumes)** — region-local block storage. Mount one into a Service with `MountVolume({ path, sizeGb })`, or pass `mounts` on a Machine. A Volume attaches to **one** Machine; `count: N` creates N disks, not one shared disk.
- **[Postgres](https://alchemy.run/fly/data/postgres)** — billed Managed Postgres. Bind `ConnectPostgres` on a Service; pass the connection string to Drizzle or SQL. `migrations` is the same surface as Neon.
- **[Redis](https://alchemy.run/fly/data/redis)** — Upstash Redis. Bind `ReadWriteRedis` on a Service.
- **[Tigris](https://alchemy.run/fly/data/tigris)** — S3-compatible object storage (`Fly.Bucket`). Bind `PutObject` / `GetObject` on a Service.

## Secrets

- **[Secrets](https://alchemy.run/fly/data/secrets)** — `Config.redacted` in a Service for values from `.env`. `Fly.Secret` when Fly should own a value shared across Machines. KMS keys are `SecretKey` plus `Encrypt` / `Decrypt` / `Sign` / `Verify`.

## Networking

- **[IPs & certificates](https://alchemy.run/fly/networking)** — shared or dedicated addresses (`IpAssignment`) so `{app}.fly.dev` answers over IPv4, plus ACME or uploaded TLS certificates for your own hostname.

## What are you building?

| You’re building | Reach for |
| --- | --- |
| An HTTP API on Fly | [App](https://alchemy.run/fly/compute/apps) + [Service](https://alchemy.run/fly/compute/services) |
| A Vite SPA | [`Website.Vite`](fly/frontend/vite.md) |
| An SSR site (Astro, Next, Nuxt, …) | [`Website`](fly/frontend/websites.md) |
| A sandbox that can sleep | [Sprite](https://alchemy.run/fly/compute/sprites) |
| A raw container | [Machine](https://alchemy.run/fly/compute/machines) with an `image` |
| A background worker | [Service](https://alchemy.run/fly/compute/services) with `ServerHost.run` |
| Persistent disk | [MountVolume](https://alchemy.run/fly/data/volumes) or Machine `mounts` |
| Managed Postgres | [Postgres](https://alchemy.run/fly/data/postgres) + `ConnectPostgres` |
| Upstash Redis | [Redis](https://alchemy.run/fly/data/redis) + `ReadRedis` / `WriteRedis` / `ReadWriteRedis` |
| Object storage | [Tigris](https://alchemy.run/fly/data/tigris) + `PutObject` / `GetObject` |
| N replicas behind fly.dev | Service `count`, or more [Machine](https://alchemy.run/fly/compute/machines) resources |
| Restore a disk | [VolumeSnapshot](https://alchemy.run/fly/data/volumes#snapshots) + mount `snapshotId` |
| Config / API tokens | [`Config.redacted`](https://alchemy.run/fly/data/secrets). [`Fly.Secret`](https://alchemy.run/fly/data/secrets) when Fly should own it |
| Sign / encrypt in-app | [SecretKey](https://alchemy.run/fly/data/secrets#kms-keys) + `Encrypt` / `Sign` |
| `{app}.fly.dev` over IPv4 | [IpAssignment](https://alchemy.run/fly/networking) `type: "shared_v4"` |
| Your own domain | [Certificate](https://alchemy.run/fly/networking#certificates) (ACME) |

## Where next

- [Setup](fly/setup.md) — create a Fly org, generate an API token, and connect alchemy.
- [Tutorial](https://alchemy.run/fly/tutorial/part-1) — from empty directory to an App running an HTTP Service with a mounted Volume and a secret.
- [Websites](fly/frontend/websites.md) — Vite, Astro, Next.js, and other framework sites as a Node Service.
- [Providers reference](https://alchemy.run/providers) — generated API docs for every Fly resource Alchemy ships.
