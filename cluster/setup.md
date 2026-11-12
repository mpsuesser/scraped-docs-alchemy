---
url: https://alchemy.run/cluster/setup
title: "Setup"
description: "Install the Effect Cluster provider and get Docker ready to run a cluster on your machine."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Effect Cluster needs no account of its own. You run the database and the pool on a cloud you already use, so setting up means installing the provider and getting that cloud ready.

## Add the provider

Install `@alchemy.run/effect-cluster`:

```sh
pnpm add @alchemy.run/effect-cluster@latest
```

Then add `Cluster.providers()` next to your cloud’s providers:

```typescript
import * as Alchemy from "alchemy";
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Api from "./src/Api.ts";
import RoomLive from "./src/RoomLive.ts";

export default Alchemy.Stack(
  "Rooms",
  { providers: Layer.mergeAll(Docker.providers(), Cluster.providers()) },
  Effect.gen(function* () {
    const api = yield* Api;
    return { api: api.name };
  }).pipe(Effect.provide(RoomLive)),
);
```

## Install the package for your cloud

Each cloud has its own entry point, and needs that cloud’s provider package:

| Import | Install |
| --- | --- |
| `@alchemy.run/effect-cluster/Docker` | `@alchemy.run/docker` |
| `@alchemy.run/effect-cluster/AWS` | `@alchemy.run/aws` |
| `@alchemy.run/effect-cluster/GCP` | `@alchemy.run/gcp` |
| `@alchemy.run/effect-cluster/Kubernetes` | `@alchemy.run/kubernetes` |
| `@alchemy.run/effect-cluster/Fly` | `@alchemy.run/fly` |
| `@alchemy.run/effect-cluster/Hetzner` | `@alchemy.run/hetzner` |
| `@alchemy.run/effect-cluster/Cloudflare` | `@alchemy.run/cloudflare` |

## Connect your cloud

Effect Cluster has no account of its own: the database and the pool deploy into whichever account, project or organization your cloud’s profile signs in to. Connect that cloud as its own setup describes:

| Cloud | Setup | Where a deploy goes |
| --- | --- | --- |
| Docker | [Docker setup](../docker/setup.md) | the Docker daemon on your machine |
| AWS | [AWS setup](../aws/setup.md) | [Accounts and regions](../aws/accounts-and-regions.md) |
| GCP | [GCP setup](../gcp/setup.md) | [Projects and regions](../gcp/projects-and-regions.md) |
| Kubernetes | [Kubernetes setup](../kubernetes/setup.md) | the cluster your kubeconfig points at |
| Fly | [Fly setup](../fly/setup.md) | [Organizations and tokens](../fly/organizations-and-tokens.md) |
| Hetzner | [Hetzner setup](../hetzner/setup.md) | [Projects and tokens](../hetzner/projects-and-tokens.md) |
| Cloudflare | [Cloudflare setup](../cloudflare/setup.md) | [Accounts and tokens](../cloudflare/accounts-and-tokens.md) |

The stage decides what is deployed, and the profile decides where it goes, so production gets a profile of its own:

```sh
pnpm alchemy deploy --stage prod --profile prod
```

In CI, the deploy reads the same credentials that cloud’s provider reads in CI. Effect Cluster adds none. See [CI](../environments/ci.md).

The database password and each pool’s gateway token are created by the Stack and kept in its state, so there’s nothing else to set. See [Databases](databases.md#credentials) and [Calling an entity](calling.md#the-gateway-token).

## Run a cluster on your machine

The Docker database and runners run on a Docker swarm. Turn swarm mode on once:

```sh
docker swarm init
```

Run `docker info --format '{{.Swarm.LocalNodeState}}'` to check. It prints `active`.

## Use a Postgres you already have

Runners only need a Postgres they can reach. To use one you run elsewhere, such as Neon or an existing RDS instance, pass its URL:

```typescript
import * as Cluster from "@alchemy.run/effect-cluster";
import * as Config from "effect/Config";

export const Database = Cluster.Database.fromUrl({
  url: Config.Redacted("CLUSTER_DATABASE_URL"),
});
```

See [Databases](databases.md).

Next, build chat rooms in the [tutorial](tutorial/part-1.md).
