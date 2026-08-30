---
url: https://alchemy.run/hetzner
title: "Hetzner"
description: "Build applications on Hetzner Cloud with Alchemy — Servers running your Effect programs as Services, plus volumes, networks, firewalls, load balancers, and DNS, all in one typed program."
access_date: 2026-08-30T18:54:07.274Z
current_date: 2026-08-30T18:54:07.274Z
---

An Alchemy app on Hetzner is one or more **Servers** — real VMs, booted in seconds and billed by the hour — running your Effect programs as **Services**: bundled with rolldown, copied over SSH, and supervised by systemd. No Dockerfile, no ansible, no hand-written unit files. Around them you declare the rest of the platform — volumes, private networks, firewalls, load balancers, and DNS — in the same TypeScript program.

New here? [Set up your project and API token](hetzner/setup.md), then start the [tutorial](https://alchemy.run/hetzner/tutorial/part-1).

## Compute

- **[Servers](https://alchemy.run/hetzner/compute/servers)** — the VM: pick a `serverType`, an `image`, and a `location`; attach SSH keys, networks, firewalls, volumes, and placement groups. Alchemy injects a deploy key so it can reach every server it manages.
- **[Services](https://alchemy.run/hetzner/compute/services)** — your code on a Server. An HTTP `fetch` handler (or a background process) deployed as a systemd unit; N Services can share one Server.
- **[SSH](https://alchemy.run/hetzner/compute/servers#run-commands-over-ssh)** — typed `exec` / `scp` against any managed Server, usable from deploy-time Effects.
- **Images & snapshots** — snapshot a Server with `Hetzner.Image`, look up stock images with `Hetzner.findImage`.

## Frontend

- **[Websites](hetzner/frontend/websites.md)** — Vite, Astro, Next.js, Nuxt, React Router, SolidStart, SvelteKit, TanStack Start, Waku, Octane, Foldkit, Vocs, and StaticSite. Each builds your app and deploys a systemd [Service](https://alchemy.run/hetzner/compute/services) on a [Server](https://alchemy.run/hetzner/compute/servers). Several sites can share one VM. Live URL is `http://{ipv4}:3000`.
- [`Vite`](hetzner/frontend/vite.md), [`Astro`](hetzner/frontend/astro.md), [`Next.js`](hetzner/frontend/nextjs.md), [`Nuxt`](hetzner/frontend/nuxt.md), [`React Router`](hetzner/frontend/react-router.md), [`SolidStart`](hetzner/frontend/solidstart.md), [`SvelteKit`](hetzner/frontend/sveltekit.md), [`TanStack Start`](hetzner/frontend/tanstack-start.md), [`Waku`](hetzner/frontend/waku.md), [`Octane`](hetzner/frontend/octane.md), [`Foldkit`](hetzner/frontend/foldkit.md), [`Vocs`](hetzner/frontend/vocs.md), [`StaticSite`](hetzner/frontend/static-site.md).

## Storage

- **[Volumes](https://alchemy.run/hetzner/data/volumes)** — network block storage attached to a Server. Mount one into a Service with the `MountVolume` binding and read/write it like a local directory.

## Networking

- **[Networks, Firewalls & Load Balancers](https://alchemy.run/hetzner/networking)** — private networks with subnets, stateful firewall rules applied to servers, and managed load balancers with health checks and TLS certificates.
- **Floating & Primary IPs** — stable public addresses that survive server replacement.

## DNS

- **[Zones & records](https://alchemy.run/hetzner/networking/dns)** — Hetzner DNS zones and record sets as resources, plus `ReadDns` / `WriteDns` bindings for querying and mutating records from deploy-time Effects.

## What are you building?

| You’re building | Reach for |
| --- | --- |
| An HTTP API on a VM | [Server](https://alchemy.run/hetzner/compute/servers) + [Service](https://alchemy.run/hetzner/compute/services) |
| A Vite SPA | [`Website.Vite`](hetzner/frontend/vite.md) — pass `server` to share the VM |
| An SSR site (Astro, Next, Nuxt, …) | [`Website`](hetzner/frontend/websites.md) |
| A background worker / daemon | [Service](https://alchemy.run/hetzner/compute/services) with `host.run(...)` — no port, no HTTP |
| Persistent state on disk | [Volume + MountVolume](https://alchemy.run/hetzner/data/volumes) |
| A load-balanced, TLS-terminated app | [Load Balancer + Certificate](https://alchemy.run/hetzner/networking#load-balancer) |
| Servers talking privately | [Network](https://alchemy.run/hetzner/networking#private-networks) + `usePrivateIp` targets |
| Locking down public access | [Firewall](https://alchemy.run/hetzner/networking#firewalls) |
| Your own domain | [Zone + RecordSet](https://alchemy.run/hetzner/networking/dns) |
| A stable IP across replacements | Primary IP / Floating IP — see [Networking](https://alchemy.run/hetzner/networking#floating--primary-ips) |
| Golden images | `Hetzner.Image` snapshots — see [Servers](https://alchemy.run/hetzner/compute/servers#snapshots) |

## Where next

- [Setup](hetzner/setup.md) — create a Hetzner project, generate an API token, and connect alchemy.
- [Tutorial](https://alchemy.run/hetzner/tutorial/part-1) — from empty directory to a Server running an HTTP Service with a mounted Volume and a firewall.
- [Websites](hetzner/frontend/websites.md) — Vite, Astro, Next.js, and other framework sites as a systemd unit.
- [Providers reference](https://alchemy.run/providers) — generated API docs for every Hetzner resource Alchemy ships.
