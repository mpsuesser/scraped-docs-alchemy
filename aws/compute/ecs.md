---
url: https://alchemy.run/aws/compute/ecs
title: "ECS"
description: "How ECS runs containers on Fargate through clusters, task definitions, tasks and services, and how to run a server, a one-off job or a scheduled job from an image, a Dockerfile or an Effect program."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

ECS runs containers. You describe a container, its image, CPU and memory, and ECS keeps it running. With **Fargate**, AWS also provides the machines underneath, so there are no servers to patch or scale.

A container is a whole process, so it fits work that doesn’t fit an invocation: a server that holds WebSockets or database connections open, a worker that loops forever, a job that runs for hours, a program that already ships as an image.

## Clusters, task definitions, tasks and services

A **task definition** is a blueprint: which image, how much CPU and memory, which ports, which IAM role. A **task** is a running container started from it, and it runs until its process exits. A **service** keeps a number of tasks running, replaces any that stop, and optionally puts a load balancer in front of them. All of them run in a **cluster**.

<svg viewBox="0 0 715 332" style="max-width: 822px" role="img" aria-label="A Service in a cluster keeps tasks running from a task definition. The load balancer spreads requests across them."><defs><marker id="alc-dag-arrow-y6lua2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="253" y="24" width="226" height="284" rx="10" ry="10"></rect><text fill="currentColor" x="265" y="41" text-anchor="start">Cluster</text></g> <g><rect stroke="currentColor" fill="none" x="269" y="50" width="194" height="242" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="451" y="67" text-anchor="end">Service · desiredCount: 2</text></g> <g><path d="M 169 108 L 263 108" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-y6lua2)"></path><text fill="currentColor" x="216" y="102" text-anchor="middle">runs</text></g> <g><path d="M 546 108 L 436 108" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-y6lua2)"></path></g><g><path d="M 618.5 140 L 618.5 244 L 436 244" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-y6lua2)"></path></g><g transform="translate(24, 76)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">task definition</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">image · cpu · role</text></g> <g transform="translate(285, 76)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">task</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">running</text></g> <g transform="translate(285, 212)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">task</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">running</text></g> <g transform="translate(546, 76)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">load balancer</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">HTTP</text></g></svg> <svg viewBox="0 0 408 356" style="max-width: 469px" role="img" aria-label="A Service in a cluster keeps tasks running from a task definition. The load balancer spreads requests across them."><defs><marker id="alc-dag-arrow-qp39kc" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="10" y="94" width="388" height="148" rx="10" ry="10"></rect><text fill="currentColor" x="22" y="111" text-anchor="start">Cluster</text></g> <g><rect stroke="currentColor" fill="none" x="26" y="120" width="356" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="38" y="137" text-anchor="start">Service · desiredCount: 2</text></g> <g><path d="M 110.5 74 L 110.5 114" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-qp39kc)"></path><text fill="currentColor" x="118.5" y="90" text-anchor="start">runs</text></g> <g><path d="M 110.5 282 L 110.5 216" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-qp39kc)"></path></g><g><path d="M 179 314 L 297.5 314 L 297.5 216" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-qp39kc)"></path></g><g transform="translate(42, 10)"><rect stroke="currentColor" fill="none" width="137" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="68.5" y="28" text-anchor="middle" dominant-baseline="middle">task definition</text> <text fill="currentColor" x="68.5" y="46" text-anchor="middle" dominant-baseline="middle">image · cpu · role</text></g> <g transform="translate(42, 146)"><rect stroke="currentColor" fill="none" width="137" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="68.5" y="28" text-anchor="middle" dominant-baseline="middle">task</text> <text fill="currentColor" x="68.5" y="46" text-anchor="middle" dominant-baseline="middle">running</text></g> <g transform="translate(229, 146)"><rect stroke="currentColor" fill="none" width="137" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="68.5" y="28" text-anchor="middle" dominant-baseline="middle">task</text> <text fill="currentColor" x="68.5" y="46" text-anchor="middle" dominant-baseline="middle">running</text></g> <g transform="translate(42, 282)"><rect stroke="currentColor" fill="none" width="137" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="68.5" y="28" text-anchor="middle" dominant-baseline="middle">load balancer</text> <text fill="currentColor" x="68.5" y="46" text-anchor="middle" dominant-baseline="middle">HTTP</text></g></svg>

A Service in a cluster keeps tasks running from a task definition. The load balancer spreads requests across them.

In Alchemy, `AWS.ECS.Task` is a task definition together with everything that builds its image, and `AWS.ECS.Service` keeps tasks running, with or without a Task of its own.

## Run a server from an image

Declare a cluster, then a Service that runs an image. `loadBalancer: true` puts a public Application Load Balancer in front of it:

```typescript
const cluster = yield* AWS.ECS.Cluster("AppCluster", {});

const web = yield* AWS.ECS.Service("Web", {
  cluster,
  image: "public.ecr.aws/nginx/nginx:1.27",
  port: 80,
  desiredCount: 2,
  loadBalancer: true,
});

return { url: web.url };
```

Deploy copies the image into a repository in your account, creates the task’s roles and log group, registers the task definition, and starts two tasks behind the load balancer. Without `vpcId` and `subnets`, the tasks run in your account’s default VPC. Give them a network of your own for production. See [VPC & networking](../networking.md).

## Run your own program

Give the Service `main` and an Effect, and Alchemy builds an image from your program. The program returns a `fetch` handler, and binds what it uses exactly as a Lambda Function does:

```typescript
const api = yield* AWS.ECS.Service(
  "Api",
  { cluster, main: import.meta.url, port: 3000, desiredCount: 2, cpu: 256, memory: 512, loadBalancer: true },
  Effect.gen(function* () {
    const putItem = yield* AWS.DynamoDB.PutItem(Items);
    return {
      fetch: Effect.succeed(HttpServerResponse.json({ ok: true })),
    };
  }).pipe(Effect.provide(AWS.DynamoDB.PutItemHttp)),
);
```

Each binding adds its statement to the **task role**, the role the container runs as, and its environment variables to the container.

The image comes from one of three places: `image`, an image from any registry, copied into your account; `context`, your own Dockerfile, built with your local Docker; or `main`, your Effect program, bundled into a base image, `oven/bun:1` unless you set `image`. Images are built for the architecture the task runs on. Set `runtimePlatform: { cpuArchitecture: "ARM64" }` to run on Graviton, which costs less per vCPU.

## Cache build layers

Builds from `context` or `main` reuse the local BuildKit cache of the machine they run on. A fresh CI runner has none and rebuilds every step. Keep the layers in the task’s ECR repository with `build.cache`:

```typescript
const api = yield* AWS.ECS.Service("Api", {
  cluster,
  context: "./api",
  port: 8080,
  build: { cache: true },
});
```

Builds then import and export every stage’s layers at the repository’s `:buildcache` tag. The cache is off by default and never changes the image, so turning it on doesn’t start a deployment. Other backends take BuildKit’s cache backends, as in [Docker builds](../../docker/build-caches.md).

## Run a job to completion

A Task on its own runs until its process exits. Give it a `run` Effect instead of `fetch`:

```typescript
const reindex = yield* AWS.ECS.Task(
  "Reindex",
  { main: import.meta.url, cpu: 1024, memory: 2048 },
  Effect.gen(function* () {
    const scan = yield* AWS.DynamoDB.Scan(Items);
    return {
      run: Effect.gen(function* () {
        const page = yield* scan({});
        yield* Effect.log(\`reindexing ${page.Count} items\`);
      }),
    };
  }).pipe(Effect.provide(AWS.DynamoDB.ScanHttp)),
);
```

Start it from another runtime by binding `ECS.RunTask`, which grants permission to run that task and pass its roles:

```typescript
const runTask = yield* AWS.ECS.RunTask(cluster, reindex);
yield* runTask({
  launchType: "FARGATE",
  networkConfiguration: { awsvpcConfiguration: { subnets: network.privateSubnetIds } },
});
```

## Run a job on a schedule

`ECS.every` runs a Task on a rate or a cron expression, with the role the schedule needs to start it:

```typescript
yield* AWS.ECS.every("NightlyReindex", "cron(0 3 * * ? *)", {
  cluster,
  task: reindex,
  subnets: network.privateSubnetIds,
  securityGroups: [jobs.groupId],
});
```

See [EventBridge & Scheduler](../messaging/eventbridge.md#schedule-containers).

## Do work in the background

A container is always running, so it can do more than answer requests. Register a loop with `ServerHost`, and it runs alongside the `fetch` handler for the life of the container:

```typescript
Effect.gen(function* () {
  const host = yield* ServerHost;
  yield* host.run(Effect.log("heartbeat").pipe(Effect.repeat(Schedule.spaced("30 seconds")), Effect.asVoid));
  return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
});
```

Use it for polling, draining a queue, or a connection that stays open across requests.

## Process scope and request scope

A container is a real process. What Construction acquires, a pool or a connection, lives until the process shuts down, and its finalizers run when ECS stops the task gracefully. Each request still gets its own `Scope`, closed when the response is sent.

This is the main difference from Lambda, where an environment can be frozen between invocations and its finalizers only get a moment at shutdown. See [Instance scope vs request scope](../../infrastructure-as-effects/runtime.md#instance-scope-vs-request-scope).

## Deploy changes

Each deploy registers a new revision of the task definition. The Service then replaces its tasks with ones from the new revision, a few at a time, so it keeps serving. Changes to the count, the network, the load balancer and deployment settings apply the same way.

A few changes can’t be applied to a running service, like its name or its cluster. Those replace the Service.

## Pay less for workers

Tasks cost the same whether busy or idle. For work that can be interrupted, run on **Fargate Spot**, AWS’s spare capacity at a steep discount, with a share on regular Fargate so the service never disappears entirely:

```typescript
capacityProviderStrategy: [
  { capacityProvider: "FARGATE_SPOT", weight: 3 },
  { capacityProvider: "FARGATE", weight: 1, base: 1 },
],
```

A Spot task gets two minutes’ notice before it’s stopped, so keep its work resumable.

## Isolate each task in its own VM

Tasks that share an instance share its kernel. To run code you don’t trust, like a tenant’s container or an agent’s generated code, give each task its own [Nitro Sandbox](nitro-sandboxes.md): a lightweight VM that the Nitro hypervisor isolates from the instance and from every other task on it.

Sandboxes don’t run on Fargate. They run on **Managed Instances**, EC2 instances that ECS launches, patches and replaces for you. Give the cluster a capacity provider whose instances can run sandboxes:

```typescript
const capacity = yield* AWS.ECS.CapacityProvider("Capacity", {
  cluster: cluster.clusterName,
  managedInstancesProvider: {
    instanceLaunchTemplate: {
      networkConfiguration: { subnets: network.privateSubnetIds },
      nitroEnclavesEnabled: true,
    },
  },
});
```

The capacity provider creates the role ECS assumes to launch instances, the role and instance profile the instances run as, and their security group. Pass `infrastructureRoleArn` and `ec2InstanceProfileArn` to use your own. It picks instance types that support sandboxes unless you set `instanceRequirements`.

Deploy waits until the capacity provider is `ACTIVE`. ECS launches instances only once tasks need them, and terminates them when you destroy the provider.

Then set the isolation mode on the task definition, and run it on that capacity:

```typescript
const task = yield* AWS.ECS.TaskDefinition("MyTask", {
  requiresCompatibilities: ["MANAGED_INSTANCES"],
  networkMode: "awsvpc",
  cpu: 1024,
  memory: 2048,
  trustedExecutionConfiguration: { isolationMode: "ISOLATED_SANDBOX" },
  containerDefinitions: [{ name: "app", image: "ghcr.io/acme/app:1", essential: true }],
});

yield* AWS.ECS.Service("MyService", {
  cluster,
  task,
  desiredCount: 2,
  capacityProviderStrategy: [{ capacityProvider: capacity.name, weight: 1 }],
  vpcId: network.vpcId,
  subnets: network.privateSubnetIds,
});
```

An `AWS.ECS.Task`, built from an image or from your own program, takes `isolation: "isolated"` instead, which sets both:

```typescript
const job = yield* AWS.ECS.Task("MyJob", {
  image: "python:3.12-slim",
  command: ["python3", "-c", "print('hello from a sandbox')"],
  cpu: 1024,
  memory: 2048,
  isolation: "isolated",
});
```

To start one on demand with `AWS.ECS.RunTask` rather than keep a Service running, see [Run a task with ECS alone](ecs-sandboxes.md#run-a-task-with-ecs-alone).

Each running task reports `trustedExecutionSummary.isolationMode` as `ISOLATED_SANDBOX`. For `CONFIDENTIAL_SANDBOX`, where the instance can’t read the task’s memory either, add an `attestationPolicy` that names the containers allowed to run. See [Isolated or confidential](nitro-sandboxes.md#isolated-or-confidential).

Each task boots its own VM, so it starts slower than a shared one. An instance can hold many sandboxes: their vCPUs can add up to five times its own, and their memory must fit in its memory.

This is ECS as AWS provides it: a task in its own VM that you start, stop and describe. `AWS.ECS.CreateSandbox` is Alchemy’s construct on top. It starts a sandbox per user or session while your app runs, and hands back a typed handle that runs commands, uses files and calls ports inside. See [ECS Sandboxes](ecs-sandboxes.md).
