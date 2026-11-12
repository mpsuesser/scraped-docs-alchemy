---
url: https://alchemy.run/docker/services
title: "Services"
description: "Run replicated containers on a swarm with Docker.Service — publish ports through the routing mesh, roll out new images with automatic rollback, connect services on overlay networks, mount Swarm secrets, and place tasks on nodes."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Swarm **service** asks for a number of identical containers, its **replicas**, and the swarm keeps that many **tasks** running, replacing any that stop. Changing a service doesn’t replace it. The swarm **rolls** its tasks over to the new version a few at a time, and rolls back if the new ones fail.

<svg viewBox="0 0 633 184" style="max-width: 728px" role="img" aria-label="With order: start-first and parallelism 1, the swarm starts each new task before stopping its old one, one task at a time. The service never drops below two replicas."><line stroke-opacity="0.2" stroke="currentColor" x1="195.4" y1="24" x2="195.4" y2="178"></line><g><rect stroke="currentColor" fill="none" x="53" y="33" width="576" height="68" rx="8"></rect><text fill="currentColor" x="4" y="65">web.1</text> <text fill="currentColor" x="4" y="79">task</text> <g><rect stroke="currentColor" fill="none" x="61" y="40" width="201.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="161.8" y="56" text-anchor="middle">web:1.0.0</text></g> <g><rect stroke="currentColor" fill="none" x="195.4" y="70" width="425.6" height="24" rx="5"></rect><text fill="currentColor" x="408.20000000000005" y="86" text-anchor="middle">web:1.1.0</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="104" x2="633" y2="104"></line><rect stroke="currentColor" fill="none" x="53" y="107" width="576" height="68" rx="8"></rect><text fill="currentColor" x="4" y="139">web.2</text> <text fill="currentColor" x="4" y="153">task</text> <g><rect stroke="currentColor" fill="none" x="61" y="114" width="403.2" height="24" rx="5"></rect><text fill="currentColor" x="262.6" y="130" text-anchor="middle">web:1.0.0</text></g> <g><rect stroke="currentColor" fill="none" x="397" y="144" width="224" height="24" rx="5"></rect><text fill="currentColor" x="509" y="160" text-anchor="middle">web:1.1.0</text></g></g> <text fill="currentColor" x="195.4" y="18" text-anchor="middle">deploy</text></svg> <svg viewBox="0 0 330 204" style="max-width: 379px" role="img" aria-label="With order: start-first and parallelism 1, the swarm starts each new task before stopping its old one, one task at a time. The service never drops below two replicas."><line stroke-opacity="0.2" stroke="currentColor" x1="76.32" y1="24" x2="76.32" y2="198"></line><g><rect stroke="currentColor" fill="none" x="0" y="51" width="326" height="60" rx="8"></rect><text fill="currentColor" x="0" y="44">web.1 <tspan fill="currentColor" dx="8">task</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="114.47999999999999" height="24" rx="5"></rect><text fill="currentColor" x="57.239999999999995" y="70" text-anchor="middle">web:1.0.0</text></g> <g><rect stroke="currentColor" fill="none" x="76.32" y="84" width="241.68" height="24" rx="5"></rect><text fill="currentColor" x="197.16" y="100" text-anchor="middle">web:1.1.0</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="114" x2="330" y2="114"></line><rect stroke="currentColor" fill="none" x="0" y="135" width="326" height="60" rx="8"></rect><text fill="currentColor" x="0" y="128">web.2 <tspan fill="currentColor" dx="8">task</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="138" width="228.95999999999998" height="24" rx="5"></rect><text fill="currentColor" x="114.47999999999999" y="154" text-anchor="middle">web:1.0.0</text></g> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="168" width="127.20000000000002" height="24" rx="5"></rect><text fill="currentColor" x="254.39999999999998" y="184" text-anchor="middle">web:1.1.0</text></g></g> <text fill="currentColor" x="76.32" y="18" text-anchor="middle">deploy</text></svg>

With order: start-first and parallelism 1, the swarm starts each new task before stopping its old one, one task at a time. The service never drops below two replicas.

## Run a service

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
      image: "ghcr.io/acme/web:1.0.0",
      replicas: 2,
      ports: [{ external: 8080, internal: 80 }],
    });

    return { service: web.name };
  }),
);
```

`image` takes a reference or a [`Docker.Image`](build-images.md) resource. A published port listens on every node, and the swarm’s **routing mesh** sends each connection to a healthy task, wherever it runs. Set `mode: "host"` on a port to publish it only on the nodes that run a task. To run your own Effect program instead of an image, see [Effectful services](effectful-services.md).

## Roll out a new image

```typescript
const web = yield* Docker.Service("web", {
  context: swarm,
  image: "ghcr.io/acme/web:1.0.0",
  image: "ghcr.io/acme/web:1.1.0",
  replicas: 2,
  updateConfig: {
    parallelism: 1,
    delay: "10s",
    order: "start-first",
    failureAction: "rollback",
  },
});
```

The deploy runs `docker service update`, and the swarm rolls the tasks over as `updateConfig` says: `parallelism` tasks at a time, `delay` between batches, new before old with `start-first`. If the new tasks fail, `failureAction: "rollback"` returns the service to its old version, and the deploy fails with `DockerServiceUpdateFailed`. `rollbackConfig` sets how the rollback itself proceeds.

Image, replicas, environment, labels, networks, ports, mounts, secrets, configs, constraints, healthcheck and policies all roll out this way. Changing the name, context, `command`, `args`, `endpointMode`, placement preferences or `readOnlyRootFs` replaces the service, removing the old one first.

## Keep tasks healthy

```typescript
const web = yield* Docker.Service("web", {
  context: swarm,
  image: "ghcr.io/acme/web:1.0.0",
  healthcheck: { cmd: ["CMD", "wget", "-qO-", "http://localhost/health"], interval: "10s", retries: 3 },
  restartPolicy: { condition: "on-failure", delay: "5s", maxAttempts: 5 },
  stopGracePeriod: "30s",
});
```

An unhealthy task is replaced, and a rollout waits for new tasks to pass their check. `restartPolicy` decides when a stopped task is restarted, and `stopGracePeriod` how long a task has to exit before it’s killed.

## Connect services on an overlay network

An **overlay** network spans every node of the swarm:

```typescript
const network = yield* Docker.Network("app-net", {
  context: swarm,
  driver: "overlay",
});

const db = yield* Docker.Service("db", {
  context: swarm,
  image: "postgres:18-alpine",
  networks: [{ name: network.name, aliases: ["postgres"] }],
});

const api = yield* Docker.Service("api", {
  context: swarm,
  image: "ghcr.io/acme/api:latest",
  networks: [network.name],
  environment: { DATABASE_HOST: "postgres" },
});
```

Services on it reach each other by service name or alias, on any node. The name resolves to a virtual IP that balances across the service’s tasks; `endpointMode: "dnsrr"` returns the task addresses instead. `attachable: true` on the network lets plain containers join it as well.

## Mount secrets and configs

A **Swarm secret** is a value the managers store encrypted and give only to the services that mount it, as a file under `/run/secrets/`. A **config** is the same for non-secret files. Create them on a manager first:

```sh
printf '%s' "$DB_PASSWORD" | docker secret create db-password -
docker config create app-config ./config.yaml
```

Then mount them:

```typescript
const app = yield* Docker.Service("app", {
  context: swarm,
  image: "ghcr.io/acme/app:latest",
  secrets: [{ source: "db-password", target: "db_password", mode: 0o400 }],
  configs: [{ source: "app-config", target: "/etc/app/config.yaml" }],
});
```

The app reads `/run/secrets/db_password`. A plain `environment` value, `Redacted` or not, is stored in the service’s definition, where `docker service inspect` shows it, so prefer a Swarm secret for passwords on a shared swarm.

## Place tasks on nodes

```typescript
const app = yield* Docker.Service("app", {
  context: swarm,
  image: "ghcr.io/acme/app:latest",
  replicas: 4,
  placement: {
    constraints: ["node.role==worker", "node.labels.zone==eu"],
    preferences: ["spread=node.labels.zone"],
    maxReplicasPerNode: 2,
  },
});
```

`constraints` limit which nodes run the tasks, `preferences` spread them across a label’s values, and `maxReplicasPerNode` caps how many share one node.

## Volumes on a swarm

`volumes` mounts a named volume or a host path into each task. A `local` volume lives on one node, so each node that runs a task gets its own, empty copy. For a database, keep its service on one node with a constraint, or use a volume driver that shares storage between nodes.
