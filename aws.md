---
url: https://alchemy.run/aws
title: "AWS"
description: "Build AWS applications with Alchemy — a runtime (usually Lambda) plus typed resources, wired together by bindings that mint least-privilege IAM policies."
access_date: 2026-08-03T19:43:15.086Z
current_date: 2026-08-03T19:43:15.086Z
---

An Alchemy app on AWS is one **runtime** — usually a Lambda Function — plus the **resources** it talks to: tables, buckets, queues, streams. You wire them together with typed **bindings**: call `S3.PutObject(bucket)` in your code and Alchemy attaches the matching least-privilege IAM statement to the function’s role. The code is the policy.

New here? [Set up credentials](aws/setup.md) first, then deploy your first [Lambda](aws/compute/lambda.md).

## Pick your runtime

- **[Lambda](aws/compute/lambda.md)** — serverless, event-driven functions with public Function URLs. The primary documented path: every resource and event source in this section is shown running on Lambda.
- **[ECS](aws/compute/ecs.md)** — long-running containers on Fargate. Alchemy bundles your Effect program into a Docker image and runs it as a Cluster + Service + Task.
- **[EKS](aws/compute/eks.md)** — managed Kubernetes (Auto Mode). Deployments, Jobs, raw manifests, and Helm charts in the same TypeScript program as the cluster — no YAML, no kubectl.
- **[EC2](aws/compute/ec2.md)** — full-control virtual machines, with the complete VPC networking toolkit around them.

Not sure which? See [Choosing a runtime](aws/compute/choosing-a-runtime.md).

## Data

- **[DynamoDB](aws/data/dynamodb.md)** — key/value tables with `GetItem` / `PutItem` bindings and change-data-capture Streams.
- **[S3](aws/data/s3.md)** — object storage with `GetObject` / `PutObject` bindings and bucket event notifications.
- **[RDS & Aurora](aws/data/rds.md)** — managed Postgres/MySQL: the `Aurora` helper stands up the whole cluster in one call, with a `Connect` binding or the Data API at runtime.

## Messaging & events

- **[SQS](aws/messaging/sqs.md)** — queues with a `SendMessage` binding and a `Stream` -shaped Lambda consumer.
- **[SNS](aws/messaging/sns.md)** — pub/sub topics with a `Publish` binding and fan-out to queues and functions.
- **[EventBridge & Scheduler](aws/messaging/eventbridge.md)** — event buses, rules, and cron/rate schedules that invoke your functions.
- **[Kinesis](aws/messaging/kinesis.md)** — ordered, sharded data streams with a `PutRecord` binding and the same `Stream` consumer surface.

## Email

- **[Receiving inbound email](https://alchemy.run/aws/email/receiving)** — SES receipt rule sets and rules that store mail in S3, fan out to SNS, invoke a Lambda, or bounce it. Region-limited to `us-east-1`, `us-west-2`, and `eu-west-1`.

## Security & secrets

- **[Secrets & env](aws/security/secrets-env.md)** —.env values via Config, Secrets Manager when the secret is shared, generated, or rotated.

## Observability

- **[CloudWatch](aws/observability/cloudwatch.md)** — dashboards and metric alarms, declared in the same Stack as the resources they watch.

## Frontend & networking

- **[Websites](aws/frontend/websites.md)** — the frontend block: static sites and built Vite apps on S3 + CloudFront as a single `StaticSite` resource.
- **[VPC & networking](aws/networking.md)** — the `Network` helper and the VPC primitives, for when ECS, EKS, or EC2 needs explicit networking.

## What are you building?

| You’re building | Reach for |
| --- | --- |
| An HTTP API | [Lambda](aws/compute/lambda.md) + Function URL + [DynamoDB](aws/data/dynamodb.md) |
| A typed HTTP API | [Effect HTTP API on Lambda](aws/apis/effect-http-api.md) |
| A typed API for external clients | [Effect RPC on Lambda](aws/apis/effect-rpc.md) |
| Drive a MicroVM from a Lambda (internal RPC) | [MicroVMs](aws/compute/microvms.md) + [Schemaless RPC](apis/schemaless.md) |
| An ML training fleet | [HyperPod](aws/compute/hyperpod.md) — Slurm or EKS orchestrated, with task governance |
| A REST API with stages/custom domains | [API Gateway](aws/apis/api-gateway.md) |
| Your own domain on a site or API | [Custom domains with Route53 + ACM](aws/networking/custom-domains.md) |
| A static site | [Deploy a static site](aws/frontend/static-site.md) on S3 + CloudFront |
| An event pipeline | [Kinesis](aws/messaging/kinesis.md) → Lambda |
| Background jobs | [SQS](aws/messaging/sqs.md) + Lambda |
| Scheduled jobs | [EventBridge Scheduler](aws/messaging/eventbridge.md) → Lambda |
| A Postgres database | [RDS & Aurora](aws/data/rds.md) |
| Kubernetes workloads or Helm charts | [EKS](aws/compute/eks.md) |
| Object processing | [S3 events](aws/messaging/s3-events.md) |
| Inbound email pipelines | [SES email receiving](https://alchemy.run/aws/email/receiving) → S3 / SNS / Lambda |
| Change data capture | [DynamoDB Streams](aws/messaging/dynamodb-streams.md) |
| API keys or credentials for a function | [Secrets & env](aws/security/secrets-env.md) |

## Where next

- [Setup](aws/setup.md) — install Alchemy and connect your AWS account.
- [Deploy a static site](aws/frontend/static-site.md) and [Effect HTTP API on Lambda](aws/apis/effect-http-api.md) — end-to-end guides for the two most common app shapes.
- [Providers reference](https://alchemy.run/providers) — generated API docs for every AWS resource Alchemy ships, well beyond the blocks documented here (CloudFront, Route53, IAM, and more).
