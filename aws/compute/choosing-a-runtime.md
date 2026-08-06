---
url: https://alchemy.run/aws/compute/choosing-a-runtime
title: "Choosing a runtime"
description: "Lambda vs ECS vs EKS vs EC2 for Alchemy apps — cold starts, cost shape, packaging, and how much of each Alchemy currently covers."
access_date: 2026-08-06T07:23:05.654Z
current_date: 2026-08-06T07:23:05.654Z
---

Every Alchemy app on AWS needs somewhere for its code to run.
Alchemy ships four runtimes, all driven by the same pattern —
bundle an Effect program, deploy it as a resource, bind building
blocks to it:

- **Lambda** — serverless functions. **Use this by default.**
- **ECS** — long-running containers on Fargate.
- **EKS** — containers as Kubernetes workloads on a managed
  Auto Mode cluster.
- **EC2** — virtual machines you fully control.

## At a glance

|                  | Lambda                                     | ECS                                        | EKS                                          | EC2                                      |
| ---------------- | ------------------------------------------ | ------------------------------------------ | -------------------------------------------- | ---------------------------------------- |
| Execution model  | Per-request, event-driven                  | Always-on containers                       | Kubernetes objects (Deployments, Jobs)       | Always-on machine                        |
| Cost shape       | Pay per invocation; scales to zero         | Pay per running task                       | Control plane + Auto Mode nodes              | Pay per running instance                 |
| Startup          | Cold starts (ms–s), then warm              | Task launch (tens of seconds), then steady | Cluster create (~10 min), Pods in seconds    | Instance boot (minutes), then steady     |
| Packaging        | Bundled zip (Rolldown)                     | Docker image, built + pushed for you       | Docker image, built + pushed for you         | Bundled program on an AMI you choose     |
| Networking       | None required (Function URL)               | VPC required (awsvpc), optional ALB        | VPC required, `LoadBalancer` Services        | VPC required, you own every primitive    |
| Alchemy coverage | Fully documented                           | Cluster/Service/Task resources             | Cluster/Deployment/Job/Manifest/Helm         | Instance + full VPC primitives           |

## Lambda — the default

Serverless, event-driven, and the runtime this documentation is
written around. You get the [`Function`](https://alchemy.run/providers/aws/lambda/function)
resource with a public Function URL, typed bindings that mint
least-privilege IAM policies from your call sites, and
`Stream`-shaped event sources for SQS, Kinesis, DynamoDB
Streams, S3 notifications, SNS, and EventBridge. Cold starts and
the 15-minute execution cap are the trade; per-request pricing
and zero idle cost are the payoff.

Choose Lambda unless you have a specific reason not to.

→ [Lambda](lambda.md)

## ECS — long-running containers

When your workload doesn't fit a request/response window — a
WebSocket server, a worker that holds connections open, anything
that should be always-on — run it as a container. Alchemy models
ECS with three resources:

- [`Cluster`](https://alchemy.run/providers/aws/ecs/cluster) — an ECS cluster for
  running tasks and services.
- [`Task`](https://alchemy.run/providers/aws/ecs/task) — bundles an inline Effect
  program, builds and pushes a Docker image to a generated ECR
  repository, provisions task + execution IAM roles and a
  CloudWatch log group, and registers a Fargate task definition.
  Tasks can serve HTTP directly and accept the same
  binding contract (env + IAM policy statements) as Lambda.
- [`Service`](https://alchemy.run/providers/aws/ecs/service) — keeps a task
  definition running with awsvpc networking; set `public: true`
  and Alchemy provisions a public ALB + listener + target group
  for you.

You pay for running tasks whether or not they're serving
traffic, and you bring a VPC (subnets + security groups) — the
[`Network`](https://alchemy.run/providers/aws/ec2/network) helper builds one in a
single call.

→ [ECS](ecs.md)

## EKS — when your platform is Kubernetes

When the workload is container-shaped but you want Kubernetes
itself — Helm charts, operators, raw manifests, or teams already
speaking Deployments and Jobs — Alchemy targets **EKS Auto Mode**,
where AWS manages the control plane, nodes, storage, and
load-balancer integration. Alchemy models it with:

- [`Cluster`](https://alchemy.run/providers/aws/eks/cluster) — the control plane;
  `compute: "auto"` turns on Auto Mode with sensible defaults.
- [`Deployment`](https://alchemy.run/providers/kubernetes/deployment) — a replicated
  Kubernetes server (the Kubernetes analog of `AWS.ECS.Service`)
  with the same three image sources as ECS: a registry `image`, a
  Dockerfile `context`, or an inline Effect program via `main`.
- [`Job`](https://alchemy.run/providers/kubernetes/job) — run-to-completion work, or a
  `CronJob` when you set `schedule`.
- [`Manifest`](https://alchemy.run/providers/kubernetes/manifest) and
  [`HelmChart`](https://alchemy.run/providers/kubernetes/helmchart) — any raw Kubernetes
  object or a rendered Helm chart, applied via server-side apply.

Bindings work the same as on Lambda and ECS — env vars plus IAM
policy statements, delivered through an EKS Pod Identity role —
and there is no YAML and no `kubectl` step. You pay for the
control plane and the nodes Auto Mode provisions, you bring a VPC
(the [`Network`](https://alchemy.run/providers/aws/ec2/network) helper builds one),
and the control plane takes ~10 minutes to create.

→ [EKS](eks.md)

For ML training fleets specifically — persistent, health-checked
GPU clusters rather than application containers — EKS pairs with
[SageMaker HyperPod](hyperpod.md): the HyperPod nodes
join your EKS cluster, and `Deployment`/`Job` opt onto them with
the `hyperpod` prop.

## EC2 — when you need the machine

Full-control VMs for workloads that need an OS, a GPU, custom
daemons, or just predictable dedicated capacity. The
[`Instance`](https://alchemy.run/providers/aws/ec2/instance) resource can act as a
low-level compute primitive or run a bundled long-lived Effect
program directly on the machine (including serving HTTP), and
Alchemy ships the complete networking toolkit around it:
[`Vpc`](https://alchemy.run/providers/aws/ec2/vpc),
[`Subnet`](https://alchemy.run/providers/aws/ec2/subnet),
[`SecurityGroup`](https://alchemy.run/providers/aws/ec2/securitygroup), NAT and
internet gateways, route tables, EIPs, and VPC endpoints (see
[VPC & networking](../networking.md)).

You own patching, scaling, and availability.

→ [EC2](ec2.md)

## Rule of thumb

1. Start with **Lambda**. Per-request cost, no servers, and the
   whole documented binding/event-source surface.
2. Move a workload to **ECS** when it's long-running, needs more
   than 15 minutes, or is already a container.
3. Pick **EKS** over ECS when you want Kubernetes itself —
   Helm charts, operators, raw manifests — not just a container
   runner.
4. Reach for **EC2** when you need the machine itself — kernel
   access, GPUs, custom networking, or software that expects a
   host.

## Where next

- [Lambda](lambda.md) — deploy a function with a public URL.
- [AWS overview](../../aws.md) — resources and recipes.
- [Providers reference](https://alchemy.run/providers) — generated docs for every
  ECS, EKS, and EC2 resource.
