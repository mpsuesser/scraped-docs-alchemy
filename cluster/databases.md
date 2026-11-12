---
url: https://alchemy.run/cluster/databases
title: "Databases"
description: "The Postgres an Effect Cluster coordinates through — what runners keep in it, and how to run one on any cloud or connect to one you have."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

Effect Cluster has no coordinator service. Runners coordinate through a Postgres **database**: each registers itself, takes locks on the shards it owns, and stores persisted messages and workflow state there.

<svg viewBox="0 0 488 204" style="max-width: 561px" role="img" aria-label="Runners never talk to a coordinator: the database is where they agree."><defs><marker id="alc-dag-arrow-qizn08" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 204 56 C 241 56, 241 102, 278 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-qizn08)"></path></g><g><path d="M 204 148 C 241 148, 241 102, 278 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-qizn08)"></path></g><g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="180" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="90" y="28" text-anchor="middle" dominant-baseline="middle">Runner 1</text> <text fill="currentColor" x="90" y="46" text-anchor="middle" dominant-baseline="middle">shards 1–150</text></g> <g transform="translate(24, 116)"><rect stroke="currentColor" fill="none" width="180" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="90" y="28" text-anchor="middle" dominant-baseline="middle">Runner 2</text> <text fill="currentColor" x="90" y="46" text-anchor="middle" dominant-baseline="middle">shards 151–300</text></g> <g transform="translate(284, 70)"><rect stroke="currentColor" fill="none" width="180" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="90" y="28" text-anchor="middle" dominant-baseline="middle">Database</text> <text fill="currentColor" x="90" y="46" text-anchor="middle" dominant-baseline="middle">runners, locks, messages</text></g></svg>

Runners never talk to a coordinator: the database is where they agree.

## A database Layer

Each cloud’s `Database` is a Layer that runs Postgres on that cloud, on a private network its pools join:

```typescript
import * as ClusterAWS from "@alchemy.run/effect-cluster/AWS";
import * as Layer from "effect/Layer";

export const Database = ClusterAWS.Database("Cluster");

export const Default = ClusterAWS.EcsPool("Default").pipe(Layer.provide(Database));
```

Give it to every pool in the cluster. A pool on a different cloud than the database can’t join its network, and the deploy says so.

## One you already have

`Cluster.Database.fromUrl` uses any Postgres the pool can reach:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Config from "effect/Config";

export const Database = Cluster.Database.fromUrl({
  url: Config.Redacted("CLUSTER_DATABASE_URL"),
});
```

## Credentials

Each cloud’s `Database` creates a random password for Postgres and keeps it in the Stack’s state, so there’s no password to choose or store. Every runner in a pool is given the database’s full connection URL as `CLUSTER_DATABASE_URL`. Callers outside the pool are never given it: they reach the cluster through the gateway.

With `fromUrl`, the URL is yours, read through `Config` like any other secret. Store it in your secrets provider, for each stage. See [Secret providers](../environments/secret-providers.md).

## What each cloud runs

| Cloud | Database |
| --- | --- |
| Docker | Postgres as a swarm service, on an overlay network |
| AWS | RDS Postgres, in a VPC of its own |
| GCP | Postgres on a Compute Engine instance with a persistent disk |
| Kubernetes | Postgres as a Deployment behind a ClusterIP Service |
| Fly | Postgres on a Fly Machine with a volume, on a private network |
| Hetzner | Postgres on a Server with a Volume, on a private Network |
| Cloudflare | none: each Durable Object keeps its own messages in its SQLite |

The RDS database is built for production. The others keep one Postgres on one disk, which suits development and small clusters; for more, run a managed Postgres and pass its URL to `fromUrl`.

## What’s in it

Effect Cluster creates its own tables on the first runner’s start: registered runners, shard locks, messages and their replies. Workflow runs are messages too. Losing the database loses persisted messages, unfinished workflows and their history; running entities just restart.
