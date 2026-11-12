---
url: https://alchemy.run/rivet/aws
title: "AWS"
description: "Run a Rivet control plane on EC2 or ECS, and pools of nodes on Fargate or EC2."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Host your Durable Objects in your AWS account, and the control plane runs on EC2 instances across availability zones, in a VPC of its own. Your pools run on Fargate or on EC2, and any Lambda, container or Worker calls the rooms by name.

```typescript title="src/cluster.ts"
import * as RivetAWS from "@alchemy.run/rivet/AWS";
import * as Layer from "effect/Layer";

export const Cluster = RivetAWS.Ec2ControlPlane("Rivet");

export const Default = RivetAWS.EcsPool("Default").pipe(Layer.provide(Cluster));
```

Install `@alchemy.run/aws` and finish [AWS setup](../aws/setup.md).

## The control plane

You get three engines across availability zones in a VPC of their own. Each has an EBS disk, and they share an S3 bucket. An internal Application Load Balancer spreads calls across them, and CloudFront serves it over HTTPS at a `cloudfront.net` address, through a VPC origin, so the engines have no public address and you need no domain.

```typescript title="src/cluster.ts"
export const Cluster = RivetAWS.Ec2ControlPlane("Rivet", {
  instances: 3,
  instanceType: "t3.medium",
  diskSize: 20,
});
```

A disk outlives its instance, so a replaced instance picks up the same data.

### On ECS

`EcsControlPlane` runs each engine as an ECS service on its own instance:

```typescript title="src/cluster.ts"
export const Cluster = RivetAWS.EcsControlPlane("Rivet", { cpu: 1024, memory: 2048 });
```

It takes the same options as `Ec2ControlPlane`.

## The pool

`EcsPool` runs nodes on Fargate:

```typescript title="src/cluster.ts"
export const Default = RivetAWS.EcsPool("Default", {
  replicas: 2,
  cpu: 512,
  memory: 1024,
  architecture: "ARM64",
}).pipe(Layer.provide(Cluster));
```

`Ec2Pool` runs one EC2 instance per node:

```typescript title="src/cluster.ts"
export const Default = RivetAWS.Ec2Pool("Default", { instanceType: "t3.small" }).pipe(
  Layer.provide(Cluster),
);
```

Nodes run in the control plane's VPC. With a control plane outside AWS, the pool creates a VPC of its own.

## Rivet Services

`ServicesPool` runs [Rivet Services](durable-streams.md) on Fargate, in the control plane's VPC:

```typescript title="src/cluster.ts"
export const Services = RivetAWS.ServicesPool("Services").pipe(Layer.provide(Cluster));
```

Its image is copied into ECR when you deploy.

## Call it from a Lambda

```typescript title="src/api.ts"
export default class Api extends Lambda.Function<Api>()(
  "Api",
  { main: import.meta.url, functionUrl: true },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return {
      fetch: Effect.gen(function* () {
        const visitors = yield* rooms.getByName("lobby").join();
        return yield* HttpServerResponse.json({ visitors });
      }),
    };
  }),
) {}
```

Every option, with its default, is in the reference: [Ec2ControlPlane](https://alchemy.run/providers/rivet/aws#ec2controlplane), [EcsControlPlane](https://alchemy.run/providers/rivet/aws#ecscontrolplane), [EcsPool](https://alchemy.run/providers/rivet/aws#ecspool), [Ec2Pool](https://alchemy.run/providers/rivet/aws#ec2pool) and [ServicesPool](https://alchemy.run/providers/rivet/aws#servicespool).
