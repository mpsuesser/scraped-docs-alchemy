---
url: https://alchemy.run/docker
title: "Docker"
description: "Images, containers, networks, and volumes as Stack resources, driven through your active Docker CLI context."
access_date: 2026-08-03T18:22:56.523Z
current_date: 2026-08-03T18:22:56.523Z
---

The Docker provider manages images, containers, networks, and volumes by shelling out to the `docker` CLI’s active context — Docker Desktop, a remote or SSH context, a CI daemon. There is no daemon API client and no credentials of its own, and the resources live in the same Stack as your cloud resources. It is separate from `Cloudflare.Container`; registry image references are the boundary between Docker-managed images and cloud container platforms.

New here? [Set up Docker](docker/setup.md) first.

## Resources

`RemoteImage`, `Network`, `Volume`, and `Container` compose into a local service — here, Postgres with a named volume, a network alias, a published port, and a healthcheck:

```typescript
export default Alchemy.Stack(
  "DockerPostgresExample",
  {
    providers: Layer.merge(Docker.providers(), Alchemy.RandomProvider()),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const password = yield* Alchemy.makeRandom("PostgresPassword", {
      bytes: 16,
    });

    const image = yield* Docker.RemoteImage("postgres-image", {
      name: "postgres",
      tag: "18-alpine",
      alwaysPull: false,
    });
    const network = yield* Docker.Network("app-network");
    const data = yield* Docker.Volume("postgres-data");

    const postgres = yield* Docker.Container("postgres", {
      name: "alchemy-example-postgres",
      image,
      environment: {
        POSTGRES_DB: "app",
        POSTGRES_USER: "alchemy",
        POSTGRES_PASSWORD: password,
      },
      ports: [{ external: 15432, internal: 5432 }],
      volumes: [
        { hostPath: data.name, containerPath: "/var/lib/postgresql/data" },
      ],
      networks: [{ name: network.name, aliases: ["postgres"] }],
      healthcheck: {
        cmd: ["CMD-SHELL", "pg_isready -U alchemy -d app"],
        interval: "5 seconds",
        timeout: "5 seconds",
        retries: 10,
      },
      start: true,
    });
    return { container: postgres.name, hostPort: postgres.ports };
  }),
);
```

`Container` accepts the image resource directly, `alwaysPull: false` pins the pulled tag so re-deploys are no-ops, and environment values accept `Redacted` secrets — they are passed via process env, never on the CLI. [Run local services](docker/local-services.md) walks through this composition.

`Image` builds from a Dockerfile:

```typescript
const image = yield* Docker.Image("app", {
  name: "my-app",
  tag: "latest",
  build: {
    context: "./app",
    dockerfile: "Dockerfile",
    args: { NODE_ENV: "production" },
  },
});
```

The build *is* the diff: `diff` runs `docker build` and compares the resulting `imageId` against the last deploy, and the result is memoized so plan + deploy build once — see [Build and push an image](docker/build-and-push.md) for tagging and pushing to a registry.

`Docker.inspectContainer(name)` reads live runtime details for any container by name, including the bound host ports.

## Compose with your cloud

[ECS](aws/compute/ecs.md) Tasks build their image with your local Docker — the AWS provider layers the same Docker service — and [Cloudflare Containers](cloudflare/compute/containers.md) bring your own image by consuming the registry reference you push from [Build and push an image](docker/build-and-push.md). By contrast, [MicroVMs](aws/compute/microvms.md) build server-side on AWS and need no local Docker at all.

## Reference

[Container](https://alchemy.run/providers/docker/container) · [Image](https://alchemy.run/providers/docker/image) · [Network](https://alchemy.run/providers/docker/network) · [RemoteImage](https://alchemy.run/providers/docker/remoteimage) · [Volume](https://alchemy.run/providers/docker/volume)
