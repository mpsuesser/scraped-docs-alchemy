---
url: https://alchemy.run/gcp
title: "GCP"
description: "Build Google Cloud applications with Alchemy — a runtime (usually Cloud Run) plus typed resources, wired together by bindings that grant least-privilege IAM."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

An Alchemy app on Google Cloud is one **runtime**, usually a Cloud Run service, plus the **resources** it uses: databases, buckets, topics, secrets. You connect them with typed **bindings**. Call `GCP.Firestore.ReadWriteDatabase(db)` in your code and Alchemy gives the service its own service account with exactly that role on that database. The code is the IAM policy.

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends GCP.Function<Api>()(
  "Api",
  { main: import.meta.url, invokerIamDisabled: true },
  Effect.gen(function* () {
    const links = yield* GCP.Firestore.Database("Links", {
      type: "FIRESTORE_NATIVE",
    });
    const db = yield* GCP.Firestore.ReadWriteDatabase(links);

    return {
      fetch: Effect.gen(function* () {
        yield* db.set("links/demo", { url: "https://alchemy.run" });
        const link = yield* db.get("links/demo");
        return yield* HttpServerResponse.json(link?.fields ?? {});
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(GCP.Firestore.ReadWriteDatabaseHttp)),
) {}
```

New here? [Set up credentials](gcp/setup.md), then start the [tutorial](gcp/tutorial/part-1.md).

## Pick your runtime

- **[Cloud Run](gcp/compute/cloud-run.md)** — HTTP services that scale to zero. The default, and what every page here builds on.
- **[Cloud Run jobs](gcp/compute/jobs.md)** — batch work that runs to completion.
- **[Worker pools](gcp/compute/worker-pools.md)** — always-on queue consumers with no URL.
- **[Cloud Functions](gcp/compute/cloud-functions.md)** — the same program, built by Google, no local Docker.
- **[GKE](gcp/compute/gke.md)** — Kubernetes Deployments and Jobs on Autopilot.

Not sure? See [Choosing a runtime](gcp/compute/choosing-a-runtime.md).

## Data

- **[Firestore](gcp/data/firestore.md)** — documents as plain objects, no instance to size.
- **[Cloud Storage](gcp/data/storage.md)** — buckets with read and write clients and signed URLs.
- **[Cloud SQL](gcp/data/cloud-sql.md)** — Postgres over the Cloud SQL socket, with Drizzle.
- **[BigQuery](gcp/data/bigquery.md)** — stream rows in, query with parameters.
- **[Memorystore](gcp/data/memorystore.md)** — Redis on a private IP.

## Messaging & events

- **[Pub/Sub](gcp/messaging/pubsub.md)** — publish, then consume by push or pull, with filters and dead letters.
- **[Cloud Scheduler](gcp/messaging/scheduler.md)** — cron handlers on your service.
- **[Storage events](gcp/messaging/storage-events.md)** — run code when an object lands.
- **[Eventarc](gcp/messaging/eventarc.md)** — react to Firestore changes and other Google events.

## Frontend & APIs

- **[Websites](gcp/frontend/websites.md)** — Astro, Next.js, Nuxt, SvelteKit, Vite, and more on Cloud Run.
- **[Effect HTTP](gcp/apis/effect-http-api.md)** and **[Effect RPC](gcp/apis/effect-rpc.md)** — typed APIs with typed clients.

## AI, security, and operations

- **[Vertex AI](gcp/ai/vertex-ai.md)** — Gemini with no API key.
- **[IAM & bindings](gcp/security/iam.md)** — exactly what each binding grants.
- **[Secret Manager & KMS](gcp/security/secrets-kms.md)** — credentials and encryption.
- **[Logging & Monitoring](gcp/observability/monitoring.md)** — uptime checks and alerts in the same Stack.
- **[VPC](gcp/networking/vpc.md)** and **[Private services](gcp/networking/private-services.md)** — private IPs and service-to-service calls.

## What are you building?

| You’re building | Reach for |
| --- | --- |
| An HTTP API | [Cloud Run](gcp/compute/cloud-run.md) + [Firestore](gcp/data/firestore.md) |
| A typed API | [Effect HTTP on Cloud Run](gcp/apis/effect-http-api.md) |
| A Postgres-backed app | [Cloud SQL](gcp/data/cloud-sql.md) |
| A website | [Websites](gcp/frontend/websites.md) |
| A static site | [Static sites](gcp/frontend/static-site.md) |
| Background jobs | [Pub/Sub](gcp/messaging/pubsub.md) + a [worker pool](gcp/compute/worker-pools.md) |
| An analytics pipeline | [Pub/Sub](gcp/messaging/pubsub.md) → [BigQuery](gcp/data/bigquery.md) |
| Scheduled work | [Cloud Scheduler](gcp/messaging/scheduler.md) or a [scheduled job](gcp/compute/jobs.md#run-it-on-a-schedule) |
| Upload processing | [Storage events](gcp/messaging/storage-events.md) |
| A chat endpoint | [Vertex AI](gcp/ai/vertex-ai.md) |
| Kubernetes workloads | [GKE](gcp/compute/gke.md) |

## Out of scope

Alchemy manages infrastructure, not end-user content. APIs whose objects are a person’s data are not modelled as resources: Gmail, Forms, Calendar, Drive, Classroom, Chat, Tasks, Keep, People, Blogger, YouTube, Fitness, Apps Script, Street View, Fact Check, and Business Profile. Neither is `prodtt-sasportal`, a test copy of the SAS Portal API.

The advertising and marketing APIs (Analytics Admin, Display & Video 360, Tag Manager, Workspace Reseller, AdSense, AdSense Platform, Ad Exchange Buyer II, Campaign Manager 360, and DoubleClick Bid Manager) need a user’s OAuth consent, which a service account can’t hold. The Content API for Shopping and Apigee Registry are retired by Google.

## Where next

- [Setup](gcp/setup.md) — connect your project.
- [Tutorial](gcp/tutorial/part-1.md) — from an empty folder to CI/CD in five parts.
- [Providers reference](https://alchemy.run/providers) — generated docs for every GCP resource, well beyond the services covered here.
