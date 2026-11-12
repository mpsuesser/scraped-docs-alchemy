---
url: https://alchemy.run/docker/what-is-docker
title: "What is Docker?"
description: "The concepts behind Docker — engines and contexts, images and Dockerfiles, registries, containers, networks, volumes and Swarm — and how @alchemy.run/docker models each of them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Docker packages a program with everything it needs to run, its runtime, libraries and files, into an **image**, and runs images as isolated **containers** on a **Docker engine**. The `docker` command on your machine drives an engine, local or remote, and everything it creates lives on that engine. The same image runs the same way on your laptop, in CI and on a server, which is why most cloud platforms take images as the unit of deployment.

Take one small app: an `api` container that talks to Postgres, which keeps its data on a volume. Your `docker` CLI reaches an engine through a context. The engine holds the images, runs each one as a container, connects the two on a network where they find each other by name, and keeps the volume outside any container.

dockerCLI

dockerdengine · local or ssh://vps

app-networknetwork

postgrescontainer

apicontainer

postgres:18image

acme/apiimage · built

pgdatavolume

A context points the CLI at one engine. The engine holds images, containers, networks and volumes. There's no account and no region: the engine is the boundary, and nothing on it is visible from another engine.

## Engines and contexts

The **Docker engine** (`dockerd`) is the background service that builds images and runs containers. Docker Desktop runs one on your machine, a Linux server runs one as a system service, and a CI job usually has one ready.

The `docker` command talks to one engine at a time, and a **context** names which: the local socket, a server over SSH, or a TCP endpoint with TLS. There’s no account or token. Whoever can reach an engine’s socket can do anything on it.

```typescript
const vps = yield* Docker.Context("vps", { docker: "host=ssh://deploy@example.com" });
```

[Podman](https://podman.io/) runs the same images through a Docker-compatible CLI, so it works here too.

See [Engines & contexts](engines-and-contexts.md).

## Images and Dockerfiles

An **image** is a read-only filesystem plus the command to run. A **Dockerfile** builds it in steps, each step adds a **layer**, and a rebuild reuses every layer up to the first step whose inputs changed.

An image is named by a repository and a **tag**, like `postgres:18-alpine`. A tag can move to a new image; a **digest** (`sha256:…`) names exactly one. Build an image from a Dockerfile, or pull one that already exists:

```typescript
const api = yield* Docker.Image("api", { name: "acme/api", build: { context: "./api" } });
const postgres = yield* Docker.RemoteImage("postgres", { name: "postgres", tag: "18-alpine" });
```

See [Build images](build-images.md) and [Pull & mirror images](remote-images.md).

## Registries

A **registry** stores images so other engines can pull them: Docker Hub, GitHub’s `ghcr.io`, or a cloud’s own, like ECR. An image built on one engine exists only there until you push it. Pushing to a registry is how it reaches a server, another node, or a cloud platform.

See [Registries & credentials](registries.md) and [Images on cloud runtimes](cloud-runtimes.md).

## Containers

A **container** is a running image: a process with its own filesystem, network and resource limits, isolated from the rest of the machine. Its filesystem is a thin writable layer on top of the image, and it’s gone when the container is removed.

```typescript
const db = yield* Docker.Container("postgres", {
  image: postgres,
  ports: [{ external: 15432, internal: 5432 }],
  start: true,
});
```

A container **publishes** a port to make it reachable from the machine, here `localhost:15432`.

See [Containers](containers.md) and [Limits & security](limits-and-security.md).

## Networks

Containers on the same **network** reach each other by name. Give Postgres the alias `postgres`, and the `api` container on that network connects to `postgres:5432`, with no published port at all.

See [Networks](networks.md).

## Volumes

A **volume** is storage the engine keeps outside any container. Mount one into a container, and its data survives the container being replaced, as a database’s must.

See [Volumes](volumes.md).

## Swarm

**Swarm** is the orchestrator built into the engine. It joins engines into a cluster and runs **services**: a number of replicas of a container, replaced when they fail and updated a few at a time.

See [Swarms](swarm.md) and [Services](services.md).

## How @alchemy.run/docker models these concepts

Every concept above is a resource in your Stack: `Docker.Context`, `Docker.Image`, `Docker.RemoteImage`, `Docker.Container`, `Docker.Network`, `Docker.Volume`, `Docker.Swarm` and `Docker.Service`. Each one takes an optional `context`, and without it uses whatever engine `docker` talks to now. Alchemy labels what it creates, so it knows which containers, networks and volumes are its own, and `lookup` references ones it doesn’t manage. A `Docker.Service` can also run an Effect program you write in the same file. Cloud runtimes that build images, like [ECS](../aws/compute/ecs.md) and [Cloudflare Containers](../cloudflare/compute/containers.md), build theirs with the same local Docker.

Next, connect Alchemy to your engine in [Setup](setup.md).
