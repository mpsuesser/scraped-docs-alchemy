---
url: https://alchemy.run/aws
title: "AWS"
description: "Build AWS applications with Alchemy — a runtime (usually Lambda) plus typed resources, wired together by bindings that mint least-privilege IAM policies."
access_date: 2026-08-03T17:26:38.937Z
current_date: 2026-08-03T17:26:38.937Z
---

An Alchemy app on AWS is one **runtime** — usually a Lambda Function — plus the **resources** it talks to: tables, buckets, queues, streams. You wire them together with typed **bindings**: call `S3.PutObject(bucket)` in your code and Alchemy attaches the matching least-privilege IAM statement to the function’s role. The code is the policy.

New here? [Set up credentials](https://alchemy.run/aws/setup) first, then deploy your first [Lambda](https://alchemy.run/aws/compute/lambda).

## Pick your runtime

- **[Lambda](https://alchemy.run/aws/compute/lambda)** — serverless, event-driven functions with public Function URLs. The primary documented path: every resource and event source in this section is shown running on Lambda.
- **[ECS](https://alchemy.run/aws/compute/ecs)** — long-running containers on Fargate. Alchemy bundles your Effect program into a Docker image and runs it as a Cluster + Service + Task.
- **[EKS](https://alchemy.run/aws/compute/eks)** — managed Kubernetes (Auto Mode). Deployments, Jobs, raw manifests, and Helm charts in the same TypeScript program as the cluster — no YAML, no kubectl.
- **[EC2](https://alchemy.run/aws/compute/ec2)** — full-control virtual machines, with the complete VPC networking toolkit around them.

Not sure which? See [Choosing a runtime](https://alchemy.run/aws/compute/choosing-a-runtime).

## Data

- **[DynamoDB](https://alchemy.run/aws/data/dynamodb)** — key/value tables with `GetItem` / `PutItem` bindings and change-data-capture Streams.
- **[S3](https://alchemy.run/aws/data/s3)** — object storage with `GetObject` / `PutObject` bindings and bucket event notifications.
- **[RDS & Aurora](https://alchemy.run/aws/data/rds)** — managed Postgres/MySQL: the `Aurora` helper stands up the whole cluster in one call, with a `Connect` binding or the Data API at runtime.

## Messaging & events

- **[SQS](https://alchemy.run/aws/messaging/sqs)** — queues with a `SendMessage` binding and a `Stream` -shaped Lambda consumer.
- **[SNS](https://alchemy.run/aws/messaging/sns)** — pub/sub topics with a `Publish` binding and fan-out to queues and functions.
- **[EventBridge & Scheduler](https://alchemy.run/aws/messaging/eventbridge)** — event buses, rules, and cron/rate schedules that invoke your functions.
- **[Kinesis](https://alchemy.run/aws/messaging/kinesis)** — ordered, sharded data streams with a `PutRecord` binding and the same `Stream` consumer surface.

## Email

- **[Receiving inbound email](https://alchemy.run/aws/email/receiving)** — SES receipt rule sets and rules that store mail in S3, fan out to SNS, invoke a Lambda, or bounce it. Region-limited to `us-east-1`, `us-west-2`, and `eu-west-1`.

## Security & secrets

- **[Secrets & env](https://alchemy.run/aws/security/secrets-env)** —.env values via Config, Secrets Manager when the secret is shared, generated, or rotated.

## Observability

- **[CloudWatch](https://alchemy.run/aws/observability/cloudwatch)** — dashboards and metric alarms, declared in the same Stack as the resources they watch.

## Frontend & networking

- **[Websites](https://alchemy.run/aws/frontend/websites)** — the frontend block: static sites and built Vite apps on S3 + CloudFront as a single `StaticSite` resource.
- **[VPC & networking](https://alchemy.run/aws/networking)** — the `Network` helper and the VPC primitives, for when ECS, EKS, or EC2 needs explicit networking.

## What are you building?

| You’re building | Reach for |
| --- | --- |
| An HTTP API | [Lambda](https://alchemy.run/aws/compute/lambda) + Function URL + [DynamoDB](https://alchemy.run/aws/data/dynamodb) |
| A typed HTTP API | [Effect HTTP API on Lambda](https://alchemy.run/aws/apis/effect-http-api) |
| A typed API for external clients | [Effect RPC on Lambda](https://alchemy.run/aws/apis/effect-rpc) |
| Drive a MicroVM from a Lambda (internal RPC) | [MicroVMs](https://alchemy.run/aws/compute/microvms) + [Schemaless RPC](https://alchemy.run/apis/schemaless) |
| An ML training fleet | [HyperPod](https://alchemy.run/aws/compute/hyperpod) — Slurm or EKS orchestrated, with task governance |
| A REST API with stages/custom domains | [API Gateway](https://alchemy.run/aws/apis/api-gateway) |
| Your own domain on a site or API | [Custom domains with Route53 + ACM](https://alchemy.run/aws/networking/custom-domains) |
| A static site | [Deploy a static site](https://alchemy.run/aws/frontend/static-site) on S3 + CloudFront |
| An event pipeline | [Kinesis](https://alchemy.run/aws/messaging/kinesis) → Lambda |
| Background jobs | [SQS](https://alchemy.run/aws/messaging/sqs) + Lambda |
| Scheduled jobs | [EventBridge Scheduler](https://alchemy.run/aws/messaging/eventbridge) → Lambda |
| A Postgres database | [RDS & Aurora](https://alchemy.run/aws/data/rds) |
| Kubernetes workloads or Helm charts | [EKS](https://alchemy.run/aws/compute/eks) |
| Object processing | [S3 events](https://alchemy.run/aws/messaging/s3-events) |
| Inbound email pipelines | [SES email receiving](https://alchemy.run/aws/email/receiving) → S3 / SNS / Lambda |
| Change data capture | [DynamoDB Streams](https://alchemy.run/aws/messaging/dynamodb-streams) |
| API keys or credentials for a function | [Secrets & env](https://alchemy.run/aws/security/secrets-env) |

## Where next

- [Setup](https://alchemy.run/aws/setup) — install Alchemy and connect your AWS account.
- [Deploy a static site](https://alchemy.run/aws/frontend/static-site) and [Effect HTTP API on Lambda](https://alchemy.run/aws/apis/effect-http-api) — end-to-end guides for the two most common app shapes.
- [Providers reference](https://alchemy.run/providers) — generated API docs for every AWS resource Alchemy ships, well beyond the blocks documented here (CloudFront, Route53, IAM, and more).
