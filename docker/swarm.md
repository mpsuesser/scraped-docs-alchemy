---
url: https://alchemy.run/docker/swarm
title: "Swarms"
description: "Turn an engine into a swarm with Docker.Swarm, locally or on a server over SSH, use a swarm you didn't create, and grow it to more nodes."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

**Swarm** is the orchestrator built into the Docker engine. A swarm is a group of engines, called **nodes**, that run [services](services.md) together. **Managers** hold the swarm’s state and take commands, and every node runs **tasks**, the containers a service asks for. When a task or a whole node fails, the managers start replacements elsewhere. It’s orchestration without installing anything beyond Docker.

alchemy deployyour machine

swarmDocker.Swarm

vpsnode · manager

web.1task

web.2task

node-2node · worker

web.3task

Alchemy talks only to a manager. The swarm places a service's tasks on every node, including nodes you joined to it by hand.

## Create a swarm

```typescript
import * as Alchemy from "alchemy";
import * as Docker from "@alchemy.run/docker";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Docker.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const swarm = yield* Docker.Swarm("swarm");

    const web = yield* Docker.Service("web", {
      context: swarm,
      image: "nginx:alpine",
      replicas: 2,
      ports: [{ external: 8080, internal: 80 }],
    });

    return { service: web.name };
  }),
);
```

`Docker.Swarm` runs `docker swarm init` on the engine, which becomes a one-node swarm whose node is both manager and worker. `docker run` and `docker build` keep working on it as before. Passing the swarm as a service’s `context` deploys the service to the same engine, after the swarm exists.

## Run it on a server

Point a [context](engines-and-contexts.md) at the server, and create the swarm there:

```typescript
const vps = yield* Docker.Context("vps", {
  docker: "host=ssh://deploy@example.com",
});

const swarm = yield* Docker.Swarm("swarm", {
  context: vps,
  advertiseAddr: "10.0.0.1",
});
```

`advertiseAddr` is the address other nodes use to reach this manager, and Docker requires it when the server has more than one network interface. `listenAddr`, `defaultAddrPool` and `subnetSize` set the swarm’s own networking. All four apply only when the swarm is first created.

## Use a swarm you didn’t create

A service only needs its engine to be a swarm manager, so you can skip `Docker.Swarm` and point services at the context:

```typescript
const vps = yield* Docker.Context("vps", {
  docker: "host=ssh://deploy@example.com",
});

const web = yield* Docker.Service("web", {
  context: vps,
  image: "nginx:alpine",
  replicas: 3,
});
```

The swarm stays someone else’s. Destroying the Stack removes its services and leaves the swarm alone.

To make the Stack own an existing swarm instead, adopt it:

```typescript
const swarm = yield* Docker.Swarm("swarm").pipe(Alchemy.adopt(true));
```

Without `adopt`, a deploy to an engine that is already a swarm fails with `OwnedBySomeoneElse`.

## What destroying does

Destroying a `Docker.Swarm` runs `docker swarm leave --force`. The node leaves the swarm, and every service running on it stops. The Stack’s own services are removed first. Adopt a swarm only when the Stack should own it this way.

## Grow it to more nodes

Joining machines is done on the hosts themselves; there’s no resource for it. On the manager, print the join command:

```sh
docker swarm join-token worker
```

Run the `docker swarm join --token … 10.0.0.1:2377` it prints on each new machine. Label nodes to steer services onto them with [placement](services.md#place-tasks-on-nodes):

```sh
docker node update --label-add zone=eu node-2
```

Every node must be able to pull a service’s images, so a swarm of several nodes needs its images in a [registry](registries.md). An image built on the manager exists only there.

## Next

Run workloads on it in [Services](services.md).
