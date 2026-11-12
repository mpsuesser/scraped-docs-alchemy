---
url: https://alchemy.run/docker/containers
title: "Containers"
description: "Run an image as a Docker.Container — start it, publish ports, pass secrets through its environment, health-check it, reach the host — and know which changes recreate it."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **container** is a running image: a process with its own filesystem, network and limits, isolated from the rest of the machine. Its filesystem is a thin writable layer over the image, gone when the container is removed. By default it can reach out to the network, but nothing outside can reach in until it **publishes** a port, which maps a port on the host to one inside the container.

<svg viewBox="0 0 501 268" style="max-width: 576px" role="img" aria-label="Only published ports are reachable from the host. Containers that talk to each other don't need published ports at all: they share a network instead."><defs><marker id="alc-bd-arrow-oorivc" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="293" y="24" width="192" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="293" y="156" width="192" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="172" y1="78" x2="307" y2="78" marker-end="url(#alc-bd-arrow-oorivc)"></line><text fill="currentColor" x="241.5" y="70" text-anchor="middle">localhost:15432</text></g> <g><path stroke="currentColor" fill="none" d="M 94 102 V 204 H 289"></path><path stroke="currentColor" fill="none" d="M 286 197 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="104" y="157">nothing published</text></g> <text fill="currentColor" x="305" y="43" text-anchor="start">container</text> <text fill="currentColor" x="473" y="175" text-anchor="end">container</text> <g><rect stroke="currentColor" fill="none" x="16" y="54" width="156" height="48" rx="8"></rect><text fill="currentColor" x="94" y="75" text-anchor="middle">psql</text> <text fill="currentColor" x="94" y="91" text-anchor="middle">your machine</text></g> <g><rect stroke="currentColor" fill="none" x="311" y="54" width="156" height="48" rx="8"></rect><text fill="currentColor" x="389" y="75" text-anchor="middle">postgres</text><text fill="currentColor" x="389" y="91" text-anchor="middle">:5432 as host:15432</text></g> <g><rect stroke="currentColor" fill="none" x="311" y="186" width="156" height="48" rx="8"></rect><text fill="currentColor" x="389" y="207" text-anchor="middle">worker</text> <text fill="currentColor" x="389" y="223" text-anchor="middle">no ports</text></g></svg> <svg viewBox="0 0 360.2 394" style="max-width: 414px" role="img" aria-label="Only published ports are reachable from the host. Containers that talk to each other don't need published ports at all: they share a network instead."><defs><marker id="alc-bd-arrow-kixbkt" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="134" width="192" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="282" width="192" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="112" y1="64" x2="112" y2="160" marker-end="url(#alc-bd-arrow-kixbkt)"></line><text fill="currentColor" x="122" y="118">localhost:15432</text></g> <g><path stroke="currentColor" fill="none" d="M 190 40 H 224 V 304 H 212"></path><path stroke="currentColor" fill="none" d="M 201 297 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="232" y="176">nothing published</text></g> <text fill="currentColor" x="28" y="153" text-anchor="start">container</text> <text fill="currentColor" x="196" y="301" text-anchor="end">container</text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="37" text-anchor="middle">psql</text> <text fill="currentColor" x="112" y="53" text-anchor="middle">your machine</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="185" text-anchor="middle">postgres</text><text fill="currentColor" x="112" y="201" text-anchor="middle">:5432 as host:15432</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="333" text-anchor="middle">worker</text> <text fill="currentColor" x="112" y="349" text-anchor="middle">no ports</text></g></svg>

Only published ports are reachable from the host. Containers that talk to each other don't need published ports at all: they share a network instead.

## Run a container

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
    const web = yield* Docker.Container("web", {
      image: "nginx:alpine",
      ports: [{ external: 8080, internal: 80 }],
      start: true,
    });

    return { name: web.name, status: web.status };
  }),
);
```

`start: true` runs it. Without it, the container is created and left stopped. `image` takes a reference like `nginx:alpine`, or a [`Docker.Image`](build-images.md) or [`Docker.RemoteImage`](remote-images.md) resource. `command` replaces the image’s default command.

## Publish ports

Each mapping publishes one container port on the host:

```typescript
const api = yield* Docker.Container("api", {
  image: "ghcr.io/acme/api:latest",
  ports: [{ external: 0, internal: 3000 }],
  start: true,
});

const hostPort = api.ports["3000/tcp"];
```

`external: 0` lets Docker choose a free host port, and `ports` reports the one it chose, keyed by container port and protocol. Add `protocol: "udp"` for UDP.

## Pass secrets in the environment

`environment` takes plain strings and `Redacted` values:

```typescript
const db = yield* Docker.Container("postgres", {
  image: "postgres:18-alpine",
  environment: {
    POSTGRES_DB: "app",
    POSTGRES_PASSWORD: yield* Config.Redacted("POSTGRES_PASSWORD"),
  },
  start: true,
});
```

Only the variable names go on the `docker` command line, so a password never shows in a process listing or shell history. Docker itself stores the values, and `docker inspect` shows them to anyone with access to the engine. See [Secrets](../environments/secrets.md) for where `Config` values come from.

Docker env files work too. Later files win, and `environment` wins over all of them:

```typescript
const app = yield* Docker.Container("app", {
  image: "ghcr.io/acme/app:latest",
  envFiles: ["./config/base.env", "./config/production.env"],
  environment: { LOG_LEVEL: "info" },
});
```

Editing an env file recreates the container on the next deploy. Alchemy keeps only a hash of the files, on the container’s label, never their values.

## Check its health

```typescript
const db = yield* Docker.Container("postgres", {
  image: "postgres:18-alpine",
  healthcheck: {
    cmd: ["CMD-SHELL", "pg_isready -U postgres"],
    interval: "5 seconds",
    timeout: "5 seconds",
    retries: 10,
    startPeriod: "10 seconds",
  },
  restart: "unless-stopped",
  start: true,
});
```

Docker runs the check inside the container and marks it `healthy` or `unhealthy`. A string `cmd` runs in the container’s shell, and `["NONE"]` turns off a check the image defines. `restart` decides whether Docker restarts the container when it exits or the engine restarts: `"no"`, `"always"`, `"on-failure"` or `"unless-stopped"`.

## Reach a service on the host

A container’s `localhost` is the container itself. To reach something on your machine, like a database you run outside Docker, add Docker’s `host-gateway` name:

```typescript
const api = yield* Docker.Container("api", {
  image: "ghcr.io/acme/api:latest",
  extraHosts: ["host.docker.internal:host-gateway"],
  environment: { DATABASE_URL: "postgres://postgres@host.docker.internal:5432/app" },
  start: true,
});
```

On Linux the name resolves to the Docker bridge’s gateway. A firewall that drops incoming traffic by default, as ufw does, then lets the name resolve but times out the connection. Allow the bridge subnet, for example `sudo ufw allow from 172.16.0.0/12`.

## What a change does

Docker can’t change most of a container’s settings once it exists, so changing the image, environment, ports, volumes, limits or almost any other prop **recreates** it. The old container is removed before the new one starts, so plan for a short gap, and keep data in a [Volume](volumes.md) so it survives.

Two changes apply in place: `networks`, which connects and disconnects it, and `start`. A `Docker.Image` passed as `image` recreates the container whenever it’s rebuilt, even under the same tag.

## Read its state

Each container’s attributes, including `status` and `ports`, are outputs of the Stack. `Docker.inspectContainer` reads the same attributes fresh from the engine, for any container by name:

```typescript
const live = yield* Docker.inspectContainer("app-postgres");
```

## Use a container you didn’t create

A container with the same name that Alchemy didn’t create stops the deploy, so a Stack never takes over someone else’s by accident. To manage it from now on, adopt it:

```typescript
const db = yield* Docker.Container("postgres", {
  name: "postgres",
  image: "postgres:18-alpine",
}).pipe(Alchemy.adopt(true));
```

To refer to one without managing it, look it up. Alchemy reads it and never changes or removes it:

```typescript
const db = Docker.Container.lookup({ name: "postgres" });
```

## Next

- [Networks](networks.md) to connect containers by name.
- [Volumes](volumes.md) for data that outlives a container.
- [Limits & security](limits-and-security.md) to cap resources and drop privileges.
